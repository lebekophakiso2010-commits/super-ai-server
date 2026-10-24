from kivy.app import App
from kivy.uix.screenmanager import ScreenManager, Screen
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.button import Button
from kivy.uix.popup import Popup
from kivy.uix.scrollview import ScrollView
from kivy.uix.gridlayout import GridLayout
from kivy.uix.image import AsyncImage
from kivy.clock import Clock
import json, os, random, requests, datetime

DATA_FILE = "superai_data.json"
SERVER_URL = "https://super-ai-server-xxxx.onrender.com"
LANGUAGES = ["English","Zulu","Xhosa","Afrikaans","Sotho","French","Portuguese","Swahili","Shona","Arabic","Hindi","Spanish"]

def load_data():
    if os.path.exists(DATA_FILE):
        try:
            with open(DATA_FILE,'r') as f: return json.load(f)
        except: pass
    return {"users":{},"current_user":None,"temp_phone":None,"temp_otp":None,"temp_lang":"English","cashout":[]}
def save_data(d):
    with open(DATA_FILE,'w') as f: json.dump(d,f)

def show_pop(title,msg):
    Popup(title=title,content=Label(text=msg),size_hint=(0.8,0.35)).open()

class LangScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        box=BoxLayout(orientation='vertical',padding=20,spacing=10)
        box.add_widget(Label(text='SUPER AI\nChoose Language',font_size=26,size_hint_y=0.2))
        scroll=ScrollView(); grid=GridLayout(cols=1,spacing=10,size_hint_y=None); grid.bind(minimum_height=grid.setter('height'))
        for lg in LANGUAGES:
            b=Button(text=lg,size_hint_y=None,height=55); b.bind(on_press=lambda x,l=lg: self.set_lang(l)); grid.add_widget(b)
        scroll.add_widget(grid); box.add_widget(scroll); self.add_widget(box)
    def set_lang(self,lg):
        d=load_data(); d['temp_lang']=lg; save_data(d); self.manager.current='phone'

class PhoneScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw); self.box=BoxLayout(orientation='vertical',padding=30,spacing=15); self.add_widget(self.box)
    def on_enter(self):
        self.box.clear_widgets(); d=load_data()
        self.box.add_widget(Label(text=f"Lang: {d['temp_lang']}",font_size=18))
        self.phone=TextInput(hint_text='Phone e.g 0711234567',multiline=False,size_hint_y=0.2); self.box.add_widget(self.phone)
        b=Button(text='CONTINUE',size_hint_y=0.2); b.bind(on_press=self.send); self.box.add_widget(b)
    def send(self,_):
        ph=self.phone.text.strip()
        if len(ph)<10: show_pop('Error','Enter valid phone'); return
        if ph.startswith('0'): ph='+27'+ph[1:]
        code=str(random.randint(100000,999999)); d=load_data(); d['temp_phone']=ph; d['temp_otp']=code; save_data(d)
        show_pop('Your OTP Code',code); self.manager.current='otp'

class OTPScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=30,spacing=10); l.add_widget(Label(text='Enter OTP'))
        self.i=TextInput(multiline=False,size_hint_y=0.2); l.add_widget(self.i)
        b1=Button(text='VERIFY',size_hint_y=0.2); b1.bind(on_press=self.verify); l.add_widget(b1)
        b2=Button(text='RESEND CODE',size_hint_y=0.2); b2.bind(on_press=self.resend); l.add_widget(b2); self.add_widget(l)
    def verify(self,_):
        d=load_data()
        if self.i.text==d.get('temp_otp'):
            ph=d['temp_phone']; lg=d['temp_lang']
            if ph not in d['users']: d['users'][ph]={"phone":ph,"lang":lg,"credits":10,"pin":None,"vip":"None","balance":0}
            d['current_user']=ph; save_data(d)
            self.manager.current='setpin' if not d['users'][ph]['pin'] else 'main'
        else: show_pop('Error','Wrong OTP')
    def resend(self,_):
        d=load_data(); c=str(random.randint(100000,999999)); d['temp_otp']=c; save_data(d); show_pop('New OTP',c)

class SetPinScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=30,spacing=10); l.add_widget(Label(text='Set 4-digit PIN'))
        self.p1=TextInput(password=True,multiline=False,size_hint_y=0.2); self.p2=TextInput(password=True,multiline=False,size_hint_y=0.2)
        l.add_widget(self.p1); l.add_widget(self.p2)
        b=Button(text='SAVE PIN',size_hint_y=0.2); b.bind(on_press=self.save); l.add_widget(b); self.add_widget(l)
    def save(self,_):
        if self.p1.text==self.p2.text and len(self.p1.text)==4 and self.p1.text.isdigit():
            d=load_data(); d['users'][d['current_user']]['pin']=self.p1.text; save_data(d); self.manager.current='main'
        else: show_pop('Error','PIN must be 4 digits match')

class MainScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw); self.box=BoxLayout(orientation='vertical',padding=12,spacing=8); self.add_widget(self.box)
    def on_enter(self):
        self.box.clear_widgets(); d=load_data(); u=d['users'][d['current_user']]
        self.box.add_widget(Label(text=f"{u['phone']} | CREDITS:{u['credits']} | BAL:R{u['balance']} | {u['vip']}",size_hint_y=0.1,font_size=12))
        for txt,tgt in [("SITE 1: EDIT STUDIO PRIVATE","edit"),("SITE 2: AI CHAT PRIVATE","chat")]:
            b=Button(text=txt,size_hint_y=0.18); b.bind(on_press=lambda x,t=tgt: setattr(self.manager,'current',t)); self.box.add_widget(b)
        row1=BoxLayout(size_hint_y=0.12,spacing=5)
        row1.add_widget(Button(text='BUY CREDITS',on_press=lambda x: setattr(self.manager,'current','buy')))
        row1.add_widget(Button(text='VIP PACKS',on_press=lambda x: setattr(self.manager,'current','vip')))
        self.box.add_widget(row1)
        row2=BoxLayout(size_hint_y=0.12,spacing=5)
        row2.add_widget(Button(text='CASH OUT',on_press=lambda x: setattr(self.manager,'current','cashout')))
        row2.add_widget(Button(text='LOCK',on_press=lambda x: setattr(self.manager,'current','lock')))
        self.box.add_widget(row2)

class EditScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        box=BoxLayout(orientation='vertical',padding=10,spacing=6)
        box.add_widget(Label(text='EDIT STUDIO - PRIVATE - R0 IDLE',size_hint_y=0.08))
        self.file=TextInput(hint_text='File name',multiline=False,size_hint_y=0.08); box.add_widget(self.file)
        self.word=TextInput(hint_text='Words to add',multiline=False,size_hint_y=0.08); box.add_widget(self.word)
        self.time=TextInput(text='1',hint_text='1-6 min',multiline=False,size_hint_y=0.07); box.add_widget(self.time)
        for txt,cr in [("EDIT PHOTO 10cr",10),("EDIT VIDEO 25cr",25),("EDIT WORDS 30cr",30),("ANIMATION 6MIN 100cr",100)]:
            b=Button(text=txt,size_hint_y=0.11); b.bind(on_press=lambda x,c=cr,t=txt: self.do_edit(c,t)); box.add_widget(b)
        box.add_widget(Button(text='BACK',size_hint_y=0.1,on_press=lambda x: setattr(self.manager,'current','main'))); self.add_widget(box)
    def do_edit(self,cr,title):
        d=load_data(); u=d['users'][d['current_user']]
        if u['credits']<cr and 'GOD' not in u['vip']: show_pop('No credits','Buy credits'); self.manager.current='buy'; return
        try: m=int(self.time.text); m=max(1,min(6,m))
        except: m=1
        if 'GOD' not in u['vip']: u['credits']-=cr; save_data(d)
        show_pop('Done',f"{title} processed {m}min")

class ChatScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        self.main=BoxLayout(orientation='vertical',padding=8,spacing=5)
        self.main.add_widget(Label(text='SITE 2: AI CHAT - PRIVATE - NO LINKS',size_hint_y=0.07,font_size=12))
        self.input=TextInput(hint_text='Ask: Who is Elon Musk?',multiline=False,size_hint_y=0.1); self.main.add_widget(self.input)
        b=Button(text='SEND 2cr',size_hint_y=0.08); b.bind(on_press=self.send); self.main.add_widget(b)
        row=BoxLayout(size_hint_y=0.08,spacing=4)
        for t in ["JOKE","WORD","PLAN","EDU"]:
            btn=Button(text=t,font_size=12); btn.bind(on_press=lambda x,tt=t: self.quick(tt)); row.add_widget(btn)
        self.main.add_widget(row)
        self.scroll=ScrollView(size_hint_y=0.57); self.box=BoxLayout(orientation='vertical',spacing=8,size_hint_y=None,padding=5)
        self.box.bind(minimum_height=self.box.setter('height')); self.scroll.add_widget(self.box); self.main.add_widget(self.scroll)
        self.main.add_widget(Button(text='BACK',size_hint_y=0.07,on_press=lambda x: setattr(self.manager,'current','main')))
        self.add_widget(self.main); Clock.schedule_once(self.greet,0.5)
    def greet(self,dt):
        h=datetime.datetime.now().hour; g="Good Morning" if h<12 else "Good Afternoon" if h<18 else "Good Evening"
        self.box.clear_widgets(); self.box.add_widget(Label(text=g,size_hint_y=None,height=30))
    def quick(self,typ):
        if typ=="JOKE": self.input.text="Tell me a joke"
        elif typ=="WORD": self.input.text="Word of the day with meaning"
        elif typ=="PLAN": self.input.text="Help me plan my day today"
        else: self.input.text="Explain photosynthesis from preschool to tertiary"
        self.send(None)
    def send(self,_):
        q=self.input.text.strip()
        if not q: return
        d=load_data(); u=d['users'][d['current_user']]
        if u['credits']<2 and 'GOD' not in u['vip']: self.manager.current='buy'; return
        self.box.add_widget(Label(text=f"You: {q}",size_hint_y=None,height=30,font_size=13))
        try:
            r=requests.get(f"{SERVER_URL}/ask",params={"q":q,"lang":u['lang']},timeout=20).json()
            bio=r.get('bio',''); photos=r.get('photos',[]); vfile=r.get('video_file','')
            lbl=Label(text=bio,size_hint_y=None,text_size=(340,None),font_size=13); lbl.bind(texture_size=lambda inst,sz: setattr(inst,'height',sz[1]+20)); self.box.add_widget(lbl)
            for img in photos[:4]:
                try: self.box.add_widget(AsyncImage(source=img,size_hint_y=None,height=180))
                except: pass
            if vfile:
                try:
                    from kivy.uix.video import Video
                    vid=Video(source=vfile,state='play',size_hint_y=None,height=200,options={'allow_stretch':True})
                    self.box.add_widget(vid)
                except:
                    self.box.add_widget(Label(text="[Video playing inside app]",size_hint_y=None,height=30,font_size=11))
        except Exception as e: self.box.add_widget(Label(text=f"Error:{e}",size_hint_y=None,height=40))
        if 'GOD' not in u['vip']: u['credits']-=2; save_data(d)
        self.input.text=''

class BuyScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=15,spacing=6); l.add_widget(Label(text='BUY CREDITS - PAYMENT',size_hint_y=0.12))
        l.add_widget(Label(text='Payment: Voucher / Capitec / Shoprite / EFT',size_hint_y=0.08,font_size=11))
        self.pin=TextInput(hint_text='Enter Voucher PIN 8+ digits',multiline=False,size_hint_y=0.12); l.add_widget(self.pin)
        for c,price in [(100,50),(200,100),(500,200),(1000,350),(2000,600)]:
            b=Button(text=f"{c} Credits - R{price}",size_hint_y=0.11); b.bind(on_press=lambda x,cr=c: self.buy(cr)); l.add_widget(b)
        l.add_widget(Button(text='BACK',size_hint_y=0.1,on_press=lambda x: setattr(self.manager,'current','main'))); self.add_widget(l)
    def buy(self,cr):
        if len(self.pin.text)<8: show_pop('Error','Enter 8+ digit voucher'); return
        d=load_data(); u=d['users'][d['current_user']]; u['credits']+=cr; save_data(d); show_pop('Success',f"Added {cr} credits"); self.manager.current='main'

class VIPScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=15,spacing=8); l.add_widget(Label(text='VIP PACKAGES - PAYMENT',size_hint_y=0.15))
        for title,bonus,price in [("Lite R149 - 500cr + 5% bonus",500,149),("Pro R299 - 1500cr + 10% cashback",1500,299),("GOD R599 - Unlimited + Earn",99999,599)]:
            b=Button(text=title,size_hint_y=0.16,font_size=12); b.bind(on_press=lambda x,bo=bonus,ti=title: self.buyvip(bo,ti)); l.add_widget(b)
        l.add_widget(Button(text='BACK',size_hint_y=0.12,on_press=lambda x: setattr(self.manager,'current','main'))); self.add_widget(l)
    def buyvip(self,bonus,title):
        d=load_data(); u=d['users'][d['current_user']]; u['credits']+=bonus; u['vip']=title; save_data(d); show_pop('VIP Activated',title); self.manager.current='main'

class CashOutScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=15,spacing=8); l.add_widget(Label(text='CASH OUT - Withdraw Earnings',size_hint_y=0.15))
        self.amount=TextInput(hint_text='Amount to cash out e.g 100',multiline=False,size_hint_y=0.12); l.add_widget(self.amount)
        self.method=TextInput(hint_text='Method: Capitec / Bank / E-Wallet / Shoprite',multiline=False,size_hint_y=0.12); l.add_widget(self.method)
        self.acc=TextInput(hint_text='Account number / Phone',multiline=False,size_hint_y=0.12); l.add_widget(self.acc)
        b=Button(text='REQUEST CASH OUT',size_hint_y=0.15); b.bind(on_press=self.request); l.add_widget(b)
        l.add_widget(Button(text='BACK',size_hint_y=0.12,on_press=lambda x: setattr(self.manager,'current','main'))); self.add_widget(l)
    def request(self,_):
        try: amt=int(self.amount.text)
        except: show_pop('Error','Enter valid amount'); return
        if not self.method.text or not self.acc.text: show_pop('Error','Fill all fields'); return
        d=load_data(); u=d['users'][d['current_user']]
        if u['balance']<amt and amt>u['credits']: show_pop('Error',f"Not enough balance. You have R{u['balance']} + {u['credits']}cr"); return
        d['cashout'].append({"phone":u['phone'],"amount":amt,"method":self.method.text,"acc":self.acc.text,"date":str(datetime.datetime.now())})
        if u['balance']>=amt: u['balance']-=amt
        else: u['credits']-=amt
        save_data(d); show_pop('Cash Out Requested',f"R{amt} to {self.method.text} will be paid in 24h"); self.manager.current='main'

class LockScreen(Screen):
    def __init__(self,**kw):
        super().__init__(**kw)
        l=BoxLayout(orientation='vertical',padding=30,spacing=10); l.add_widget(Label(text='App Locked - Enter PIN'))
        self.p=TextInput(password=True,multiline=False,size_hint_y=0.2); l.add_widget(self.p)
        l.add_widget(Button(text='UNLOCK',size_hint_y=0.2,on_press=self.unlock))
        l.add_widget(Button(text='FORGOT PIN - OTP',size_hint_y=0.2,on_press=self.recover)); self.add_widget(l)
    def unlock(self,_):
        d=load_data()
        if self.p.text==d['users'][d['current_user']]['pin']: self.manager.current='main'
        else: show_pop('Error','Wrong PIN')
    def recover(self,_):
        d=load_data(); c=str(random.randint(100000,999999)); d['temp_otp']=c; save_data(d); show_pop('Recovery OTP',c); self.manager.current='otp'

class SuperAIApp(App):
    def build(self):
        sm=ScreenManager()
        sm.add_widget(LangScreen(name='lang')); sm.add_widget(PhoneScreen(name='phone'))
        sm.add_widget(OTPScreen(name='otp')); sm.add_widget(SetPinScreen(name='setpin'))
        sm.add_widget(MainScreen(name='main')); sm.add_widget(EditScreen(name='edit'))
        sm.add_widget(ChatScreen(name='chat')); sm.add_widget(BuyScreen(name='buy'))
        sm.add_widget(VIPScreen(name='vip')); sm.add_widget(CashOutScreen(name='cashout'))
        sm.add_widget(LockScreen(name='lock'))
        return sm

SuperAIApp().run()
