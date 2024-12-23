# Text-Editor-with-Undo-Redo-Functionality-
Text Editor with Undo/Redo Functionality
from flask import Flask, request, jsonify, render_template_string

app = Flask(__name__)

# Stacks to manage undo/redo operations
undo_stack = []
redo_stack = []

# HTML content for the frontend
HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text Editor with Undo/Redo</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        .editor {
            width: 80%;
            height: 300px;
            border: 1px solid #ccc;
            margin: 20px auto;
            padding: 10px;
            overflow-y: auto;
            background: #f9f9f9;
        }
        button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <h1>Text Editor with Undo/Redo</h1>
    <div class="editor" contenteditable="true" id="editor"></div>
    <div>
        <button id="undo">Undo</button>
        <button id="redo">Redo</button>
    </div>
    <script>
        const editor = document.getElementById('editor');
        const undoBtn = document.getElementById('undo');
        const redoBtn = document.getElementById('redo');

        // Save initial content
        fetch('/save', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ content: editor.innerHTML })
        });

        // Save content on input
        editor.addEventListener('input', () => {
            fetch('/save', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ content: editor.innerHTML })
            });
        });

        // Undo operation
        undoBtn.addEventListener('click', () => {
            performUndo();
        });

        // Redo operation
        redoBtn.addEventListener('click', () => {
            performRedo();
        });

        // Function to perform undo
        function performUndo() {
            fetch('/undo')
                .then(response => response.json())
                .then(data => {
                    editor.innerHTML = data.content;
                });
        }

        // Function to perform redo
        function performRedo() {
            fetch('/redo')
                .then(response => response.json())
                .then(data => {
                    editor.innerHTML = data.content;
                });
        }

        // Keyboard shortcuts
        document.addEventListener('keydown', (event) => {
            if (event.ctrlKey && event.key === 'z') {
                event.preventDefault(); // Prevent default browser undo
                performUndo();
            }
            if (event.ctrlKey && event.key === 'y') {
                event.preventDefault(); // Prevent default browser redo
                performRedo();
            }
        });
    </script>
</body>
</html>
"""

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/save', methods=['POST'])
def save():
    content = request.json.get('content')
    undo_stack.append(content)
    redo_stack.clear()  # Clear redo stack on new input
    return jsonify({"message": "Content saved successfully"})

@app.route('/undo', methods=['GET'])
def undo():
    if len(undo_stack) > 1:
        redo_stack.append(undo_stack.pop())
        return jsonify({"content": undo_stack[-1]})
    return jsonify({"content": undo_stack[-1] if undo_stack else ""})

@app.route('/redo', methods=['GET'])
def redo():
    if redo_stack:
        state = redo_stack.pop()
        undo_stack.append(state)
        return jsonify({"content": state})
    return jsonify({"content": undo_stack[-1] if undo_stack else ""})

if __name__ == '__main__':
    app.run(debug=True)

