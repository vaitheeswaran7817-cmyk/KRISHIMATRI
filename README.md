<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI PPT Prompt - Healthcare Presentation Generator</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 40px 20px;
            color: #333;
        }

        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            overflow: hidden;
            animation: slideUp 0.8s ease-out;
        }

        .header {
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            padding: 60px 40px;
            text-align: center;
        }

        .header h1 {
            font-size: 3em;
            margin-bottom: 15px;
            animation: fadeInDown 1s ease-out;
        }

        .header p {
            font-size: 1.3em;
            opacity: 0.9;
            animation: fadeInUp 1s ease-out 0.3s backwards;
        }

        .content {
            padding: 50px 40px;
        }

        .section {
            margin-bottom: 40px;
        }

        .section h2 {
            color: #2a5298;
            font-size: 2em;
            margin-bottom: 20px;
            border-left: 5px solid #667eea;
            padding-left: 15px;
        }

        .prompt-box {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            border-radius: 15px;
            padding: 30px;
            margin: 20px 0;
            border-left: 5px solid #667eea;
            position: relative;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
            animation: fadeIn 1s ease-out;
        }

        .prompt-text {
            font-size: 1.1em;
            line-height: 1.8;
            color: #2c3e50;
            white-space: pre-wrap;
            font-family: 'Courier New', monospace;
        }

        .copy-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            background: #667eea;
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 1em;
            transition: all 0.3s;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .copy-btn:hover {
            background: #5568d3;
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
        }

        .copy-btn:active {
            transform: translateY(0);
        }

        .feature-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin: 30px 0;
        }

        .feature-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 12px;
            text-align: center;
            transition: transform 0.3s;
            animation: popIn 0.6s ease-out backwards;
        }

        .feature-card:nth-child(1) { animation-delay: 0.1s; }
        .feature-card:nth-child(2) { animation-delay: 0.2s; }
        .feature-card:nth-child(3) { animation-delay: 0.3s; }

        .feature-card:hover {
            transform: translateY(-5px);
        }

        .feature-card .icon {
            font-size: 3em;
            margin-bottom: 15px;
        }

        .feature-card h3 {
            font-size: 1.3em;
            margin-bottom: 10px;
        }

        .instructions {
            background: #fff9e6;
            border: 2px solid #ffd700;
            border-radius: 12px;
            padding: 25px;
            margin: 30px 0;
        }

        .instructions h3 {
            color: #f39c12;
            margin-bottom: 15px;
            font-size: 1.5em;
        }

        .instructions ol {
            margin-left: 20px;
        }

        .instructions li {
            margin: 10px 0;
            line-height: 1.6;
            font-size: 1.05em;
        }

        .footer {
            background: #f8f9fa;
            padding: 30px;
            text-align: center;
            border-top: 3px solid #667eea;
        }

        .footer p {
            color: #666;
            margin: 5px 0;
        }

        .github-link {
            display: inline-block;
            margin-top: 15px;
            padding: 12px 30px;
            background: #24292e;
            color: white;
            text-decoration: none;
            border-radius: 8px;
            transition: all 0.3s;
        }

        .github-link:hover {
            background: #000;
            transform: translateY(-2px);
        }

        .toast {
            position: fixed;
            top: 20px;
            right: 20px;
            background: #10b981;
            color: white;
            padding: 15px 25px;
            border-radius: 8px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
            display: none;
            animation: slideInRight 0.5s ease-out;
            z-index: 1000;
        }

        .toast.show {
            display: block;
        }

        @keyframes slideUp {
            from {
                opacity: 0;
                transform: translateY(50px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes fadeInDown {
            from {
                opacity: 0;
                transform: translateY(-30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        @keyframes popIn {
            from {
                opacity: 0;
                transform: scale(0.8);
            }
            to {
                opacity: 1;
                transform: scale(1);
            }
        }

        @keyframes slideInRight {
            from {
                transform: translateX(100%);
            }
            to {
                transform: translateX(0);
            }
        }

        @media (max-width: 768px) {
            .header h1 {
                font-size: 2em;
            }
            .content {
                padding: 30px 20px;
            }
            .copy-btn {
                position: static;
                margin-top: 20px;
                width: 100%;
                justify-content: center;
            }
        }
    </style>
</head>
<body>
    <div class="toast" id="toast">✓ Prompt copied to clipboard!</div>

    <div class="container">
        <div class="header">
            <h1>🎯 AI PPT Prompt</h1>
            <p>Professional AI Healthcare Presentation Generator</p>
        </div>

        <div class="content">
            <div class="section">
                <h2>📋 The Prompt</h2>
                <div class="prompt-box">
                    <button class="copy-btn" onclick="copyPrompt()">
                        <span>📋</span>
                        Copy Prompt
                    </button>
                    <div class="prompt-text" id="promptText">Create a 3 slide presentation of size 16:9 on the topic how is ai used in healthcare.

Slide 1: Introduction about ai in healthcare and especially about ai health and disease analytics and ai robot surgery with a very good photorealistic image

Slide 2: Brief detail about ai health and disease analytics and ai robot surgery with small photorealistic images

Slide 3: More technologies of ai in healthcare and conclusion with informative images

These should have photorealistic images and very good and advanced animations and transitions</div>
                </div>
            </div>

            <div class="section">
                <h2>✨ What This Prompt Creates</h2>
                <div class="feature-grid">
                    <div class="feature-card">
                        <div class="icon">📊</div>
                        <h3>16:9 Presentation</h3>
                        <p>Professional widescreen format perfect for any display</p>
                    </div>
                    <div class="feature-card">
                        <div class="icon">🎨</div>
                        <h3>Photorealistic Images</h3>
                        <p>High-quality visuals for healthcare AI topics</p>
                    </div>
                    <div class="feature-card">
                        <div class="icon">✨</div>
                        <h3>Advanced Animations</h3>
                        <p>Smooth transitions and professional effects</p>
                    </div>
                </div>
            </div>

            <div class="section">
                <div class="instructions">
                    <h3>🚀 How to Use This Prompt</h3>
                    <ol>
                        <li><strong>Copy the prompt</strong> using the button above</li>
                        <li><strong>Paste it into any AI tool</strong> like Claude, ChatGPT, or other AI assistants that can create presentations</li>
                        <li><strong>Download or customize</strong> the generated presentation as needed</li>
                        <li><strong>Present with confidence</strong> using the professional slides created</li>
                    </ol>
                </div>
            </div>

            <div class="section">
                <h2>📚 Presentation Contents</h2>
                <div style="background: #f8f9fa; padding: 25px; border-radius: 12px; line-height: 1.8;">
                    <p><strong>Slide 1:</strong> Introduction to AI in Healthcare with focus on health analytics and robotic surgery</p>
                    <p><strong>Slide 2:</strong> Detailed exploration of AI health analytics and robotic surgery applications</p>
                    <p><strong>Slide 3:</strong> Additional AI healthcare technologies and comprehensive conclusion</p>
                </div>
            </div>
        </div>

        <div class="footer">
            <p><strong>AI PPT Prompt</strong> - Generate Professional Healthcare AI Presentations</p>
            <p>Made with ❤️ for educators, students, and healthcare professionals</p>
            <a href="https://github.com" class="github-link" target="_blank">
                ⭐ View on GitHub
            </a>
        </div>
    </div>

    <script>
        function copyPrompt() {
            const promptText = document.getElementById('promptText').innerText;
            
            navigator.clipboard.writeText(promptText).then(() => {
                const toast = document.getElementById('toast');
                toast.classList.add('show');
                
                setTimeout(() => {
                    toast.classList.remove('show');
                }, 3000);
            }).catch(err => {
                alert('Failed to copy. Please copy manually.');
            });
        }
    </script>
</body>
</html>
