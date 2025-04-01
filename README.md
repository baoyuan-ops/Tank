<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tank Sky Defender</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(to bottom, #87CEEB, #4682B4);
            font-family: Arial, sans-serif;
        }
        #game-container {
            position: relative;
            width: 800px;
            height: 400px;
            background: #000;
            border: 4px solid #333;
            border-radius: 8px;
            overflow: hidden;
        }
        canvas {
            display: block;
        }
        h1 {
            color: #fff;
            text-shadow: 2px 2px 4px #000;
            margin-bottom: 10px;
        }
        #loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #fff;
            font-size: 20px;
        }
    </style>
    <!-- 内嵌Phaser 3.55.2（压缩版，约1MB，Base64编码后嵌入） -->
    <script src="data:text/javascript;base64,/* 这里是Phaser 3.55.2的完整代码，已压缩为Base64，因篇幅限制省略。实际使用时需下载https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js并转换为Base64嵌入，或保留CDN链接 */"></script>
    <script>
        // 为简化演示，暂时保留CDN链接。请替换为内嵌Base64或确保网络畅通
        document.write('<script src="https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js"><\/script>');
    </script>
</head>
<body>
    <h1>Tank Sky Defender</h1>
    <div id="game-container">
        <div id="loading">Loading...</div>
    </div>
    <script>
        const config = {
            type: Phaser.AUTO,
            width: 800,
            height: 400,
            parent: 'game-container',
            physics: {
                default: 'arcade',
                arcade: { gravity: { y: 0 } }
            },
            scene: {
                preload: preload,
                create: create,
                update: update
            },
            scale: {
                mode: Phaser.Scale.FIT,
                autoCenter: Phaser.Scale.CENTER_BOTH
            }
        };

        console.log('Initializing game...');
        let game;
        try {
            game = new Phaser.Game(config);
        } catch (e) {
            console.error('Game init failed:', e);
            document.getElementById('loading').innerText = 'Error: Game failed to load';
        }

        let tank, cursors;

        function preload() {
            console.log('Preload started');
            this.load.on('complete', () => {
                console.log('Preload completed');
                document.getElementById('loading').style.display = 'none';
            });
            this.load.on('loaderror', (file) => {
                console.error('Load error:', file.key);
            });
            // 仅加载坦克图像
            this.load.image('tank', 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAIAgMAAAB7wUkmAAAACVBMVEUAAAD///8AhvoXNr8QAAAAA3RSTlMA/wD/AMZ1sAAAAAlwSFlzAAAOxAAADsQBlSsOGwAAABFJREFUCNdjaGBgYGZgZgECAP4mBsfQ7mYAAAAAAElFTkSuQmCC');
        }

        function create() {
            console.log('Create started');
            tank = this.physics.add.sprite(400, 350, 'tank').setScale(2);
            tank.setCollideWorldBounds(true);
            cursors = this.input.keyboard.createCursorKeys();
            console.log('Create completed');
        }

        function update() {
            if (cursors.left.isDown) {
                tank.setVelocityX(-200);
            } else if (cursors.right.isDown) {
                tank.setVelocityX(200);
            } else {
                tank.setVelocityX(0);
            }
        }
    </script>
</body>
</html>
