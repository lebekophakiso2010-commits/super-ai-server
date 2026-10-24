from flask import Flask, request, jsonify
import requests
import urllib.parse

app = Flask(__name__)

@app.route('/')
def home():
    return "SUPER AI SERVER LIVE"

@app.route('/ask')
def ask():
    q = request.args.get('q', '').strip()
    lang = request.args.get('lang', 'English')
    if not q:
        return jsonify({"bio": "", "photos": [], "video_file": ""})
    try:
        search_url = f"https://en.wikipedia.org/w/api.php?action=query&list=search&srsearch={urllib.parse.quote(q)}&format=json"
        s = requests.get(search_url, timeout=15).json()
        hits = s.get('query', {}).get('search', [])
        best = hits[0]['title'] if hits else q
        summ_url = f"https://en.wikipedia.org/api/rest_v1/page/summary/{urllib.parse.quote(best)}"
        summ = requests.get(summ_url, timeout=15).json()
        bio = summ.get('extract', 'No data found')
        title = summ.get('title', best)
        media_url = f"https://en.wikipedia.org/api/rest_v1/page/media/{best}"
        media = requests.get(media_url, timeout=15).json()
        photos = []
        video_file = ""
        for item in media.get('items', []):
            try:
                if item.get('type') == 'image' and len(photos) < 5:
                    if 'srcset' in item and item['srcset']:
                        src = item['srcset'][-1]['src']
                        if src.startswith('//'): src = 'https:' + src
                        if src.startswith('http'): photos.append(src)
                if item.get('type') == 'video' and not video_file:
                    if 'srcset' in item and item['srcset']:
                        video_file = item['srcset'][-1]['src']
            except:
                pass
        if not photos and 'thumbnail' in summ:
            th = summ['thumbnail']['source']
            if th.startswith('//'): th = 'https:' + th
            photos = [th]
        # hidden fallback video, source not shown to user
        if not video_file:
            video_file = ""
        full_bio = f"{title}\n\n{bio}\n\nLanguage: {lang}\n\nPRESCHOOL: Simple story about {title}.\nPRIMARY: {bio[:200]}\nHIGH SCHOOL: Details and facts.\nTERTIARY: Full analysis, history, impact and future of {title}."
        return jsonify({"bio": full_bio, "photos": photos, "video_file": video_file})
    except Exception as e:
        return jsonify({"bio": f"Error: {e}", "photos": [], "video_file": ""})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=10000)
