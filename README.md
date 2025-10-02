# K-aifrom flask import Flask, request, jsonify, render_template_string
import os
import openai

# Make sure you set your API key as an environment variable before running
# Example: export OPENAI_API_KEY="your_key_here" (Linux/Mac) or set in Windows)
openai.api_key = os.getenv("OPENAI_API_KEY")

app = Flask(__name__)

HTML_PAGE = """
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Public AI Chatbot</title>
<style>
body { font-family: Arial, sans-serif; margin: 20px; }
#chatbox { border: 1px solid #ccc; padding: 10px; height: 400px; overflow-y: scroll; }
#message { width: 80%; padding: 5px; }
button { padding: 5px 10px; }
.user { color: blue; }
.bot { color: green; }
</style>
</head>
<body>
<h1>AI Chatbot</h1>
<div id="chatbox"></div>
<input type="text" id="message" placeholder="Type a message..." />
<button onclick="sendMessage()">Send</button>

<script>
async function sendMessage() {
    const msgInput = document.getElementById("message");
    const message = msgInput.value;
    if (!message) return;
    const chatbox = document.getElementById("chatbox");
    chatbox.innerHTML += `<p class="user"><b>You:</b> ${message}</p>`;
    msgInput.value = "";

    const response = await fetch("/chat", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ message })
    });
    const data = await response.json();
    chatbox.innerHTML += `<p class="bot"><b>AI:</b> ${data.reply}</p>`;
    chatbox.scrollTop = chatbox.scrollHeight;
}
</script>
</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HTML_PAGE)

@app.route("/chat", methods=["POST"])
def chat():
    user_msg = request.json.get("message", "")
    if not user_msg:
        return jsonify({"reply": "Type something to chat!"})

    try:
        resp = openai.ChatCompletion.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "You are a helpful, friendly public AI chatbot."},
                {"role": "user", "content": user_msg}
            ],
            max_tokens=400,
            temperature=0.7
        )
        reply = resp.choices[0].message.content
    except Exception as e:
        reply = f"Error: {str(e)}"

    return jsonify({"reply": reply})

if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host="0.0.0.0", port=port)
