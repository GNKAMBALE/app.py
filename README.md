from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

BASE_API_URL = "https://app.ylytic.com/ylytic/test"
@app.route('/search', methods=['GET'])
def search_comments():
    search_author = request.args.get('search_author')
    at_from = request.args.get('at_from')
    at_to = request.args.get('at_to')
    like_from = request.args.get('like_from')
    like_to = request.args.get('like_to')
    reply_from = request.args.get('reply_from')
    reply_to = request.args.get('reply_to')
    search_text = request.args.get('search_text')

    
    url = f"{BASE_API_URL}?search_author={search_author}"

    
    if at_from:
        url += f"&at_from={at_from}"

    if at_to:
        url += f"&at_to={at_to}"

    
    response = requests.get(url)

    if response.status_code == 200:
        comments = response.json()

        
        filtered_comments = []

        for comment in comments:
            if (
                (like_from is None or comment['like'] >= int(like_from))
                and (like_to is None or comment['like'] <= int(like_to))
                and (reply_from is None or comment['reply'] >= int(reply_from))
                and (reply_to is None or comment['reply'] <= int(reply_to))
                and (search_text is None or search_text in comment['text'])
            ):
                filtered_comments.append(comment)

        return jsonify(filtered_comments)

    return jsonify({'error': 'Failed to fetch comments'})

if __name__ == '__main__':
    app.run(debug=True)
