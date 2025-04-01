<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tank Sky Defender</title>
    <script src="https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js"></script>
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
            },
            fps: { target: 30 } // 限制帧率，减少性能压力
        };

        let game = new Phaser.Game(config);
        let tank, planes, enemyTanks, bosses, bullets, cursors, touchX = null;
        let score = 0, lives = 3, level = 1, scoreText, livesText, levelText, background;
        let enemySpawnRate = 1500, enemySpeedMultiplier = 1, bossActive = false;

        function preload() {
            this.load.on('complete', () => {
                document.getElementById('loading').style.display = 'none';
            });
            this.load.on('loaderror', (file) => {
                console.error('Failed to load:', file.key); // 调试资源加载失败
            });
            // 使用可靠的在线资源
            this.load.image('tank', 'https://labs.phaser.io/assets/sprites/tank.png');
            this.load.image('plane', 'https://labs.phaser.io/assets/sprites/plane.png');
            this.load.image('enemyTank', 'https://labs.phaser.io/assets/sprites/enemy-tank.png');
            this.load.image('boss', 'https://labs.phaser.io/assets/sprites/boss1.png');
            this.load.image('bullet', 'https://labs.phaser.io/assets/particles/yellow.png');
            this.load.image('background', 'https://labs.phaser.io/assets/skies/desert-back.png');
            this.load.audio('shoot', 'https://labs.phaser.io/assets/audio/gunshot.mp3');
            this.load.audio('explode', 'https://labs.phaser.io/assets/audio/explosion.mp3');
        }

        function create() {
            background = this.add.tileSprite(0, 0, 800, 400, 'background').setOrigin(0, 0);
            tank = this.physics.add.sprite(100, 350, 'tank').setScale(0.5);
            tank.setCollideWorldBounds(true);

            planes = this.physics.add.group({ maxSize: 10 }); // 限制敌人数量
            enemyTanks = this.physics.add.group({ maxSize: 10 });
            bosses = this.physics.add.group({ maxSize: 1 });
            bullets = this.physics.add.group({ maxSize: 20 });

            scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '20px', color: '#fff', stroke: '#000', strokeThickness: 2 });
            livesText = this.add.text(10, 30, 'Lives: 3', { fontSize: '20px', color: '#fff', stroke: '#000', strokeThickness: 2 });
            levelText = this.add.text(700, 10, 'Level: 1', { fontSize: '20px', color: '#fff', stroke: '#000', strokeThickness: 2 });

            cursors = this.input.keyboard.createCursorKeys();
            this.input.addPointer(1);
            this.input.on('pointermove', (pointer) => { touchX = pointer.x; });
            this.input.on('pointerdown', () => shootBullet.call(this));

            startLevel.call(this);
            createCollisions.call(this);
        }

        function update() {
            background.tilePositionX += 2;

            if (cursors.left.isDown) {
                tank.setVelocityX(-200);
            } else if (cursors.right.isDown) {
                tank.setVelocityX(200);
            } else if (touchX !== null) {
                if (touchX < tank.x) tank.setVelocityX(-200);
                else if (touchX > tank.x) tank.setVelocityX(200);
                else tank.setVelocityX(0);
            } else {
                tank.setVelocityX(0);
            }

            if (Phaser.Input.Keyboard.JustDown(cursors.space)) {
                shootBullet.call(this);
            }

            if (!bossActive && planes.countActive() === 0 && enemyTanks.countActive() === 0 && score >= level * 50) {
                if (level % 5 === 0) {
                    spawnBoss.call(this);
                } else {
                    levelUp.call(this);
                }
            }
        }

        function startLevel() {
            enemySpawnRate = Math.max(500, 1500 - level * 200);
            enemySpeedMultiplier = Math.min(2, 1 + level * 0.2); // 限制最大速度
            this.time.addEvent({ delay: enemySpawnRate, callback: spawnPlane, callbackScope: this, loop: true });
            this.time.addEvent({ delay: enemySpawnRate * 1.5, callback: spawnEnemyTank, callbackScope: this, loop: true });
            levelText.setText('Level: ' + level);
            bossActive = false;
        }

        function levelUp() {
            level += 1;
            this.time.removeAllEvents();
            startLevel.call(this);
        }

        function spawnPlane() {
            if (!bossActive && planes.getLength() < planes.maxSize) {
                let plane = planes.create(800, Phaser.Math.Between(50, 150), 'plane').setScale(0.5);
                plane.setVelocityX(-150 * enemySpeedMultiplier);
            }
        }

        function spawnEnemyTank() {
            if (!bossActive && enemyTanks.getLength() < enemyTanks.maxSize) {
                let enemyTank = enemyTanks.create(800, 350, 'enemyTank').setScale(0.5);
                enemyTank.setVelocityX(-100 * enemySpeedMultiplier);
            }
        }

        function spawnBoss() {
            bossActive = true;
            this.time.removeAllEvents();
            let boss = bosses.create(700, 100, 'boss').setScale(1);
            boss.setVelocityX(-50 * enemySpeedMultiplier);
            boss.health = 50;
            this.add.text(300, 50, 'Boss Fight!', { fontSize: '30px', color: '#ff0000', stroke: '#000', strokeThickness: 2 }).setDepth(1);
        }

        function shootBullet() {
            if (bullets.getLength() < bullets.maxSize) {
                let bullet = bullets.create(tank.x, tank.y - 20, 'bullet').setScale(0.5);
                bullet.setVelocityY(-400);
                bullet.checkWorldBounds = true;
                bullet.outOfBoundsKill = true;
                this.sound.play('shoot');
            }
        }

        function hitPlane(bullet, plane) {
            bullet.destroy();
            plane.destroy();
            score += 10;
            scoreText.setText('Score: ' + score);
            this.sound.play('explode');
        }

        function hitEnemyTank(bullet, enemyTank) {
            bullet.destroy();
            enemyTank.destroy();
            score += 20;
            scoreText.setText('Score: ' + score);
            this.sound.play('explode');
        }

        function hitBoss(bullet, boss) {
            bullet.destroy();
            boss.health -= 1;
            if (boss.health <= 0) {
                boss.destroy();
                score += 100;
                scoreText.setText('Score: ' + score);
                this.sound.play('explode');
                bossActive = false;
                levelUp.call(this);
            } else {
                this.sound.play('explode');
            }
        }

        function takeDamage(tank, enemy) {
            enemy.destroy();
            lives -= 1;
            livesText.setText('Lives: ' + lives);
            this.sound.play('explode');
            if (lives <= 0) {
                this.physics.pause();
                this.add.text(300, 200, 'Game Over', { fontSize: '40px', color: '#ff0000', stroke: '#000', strokeThickness: 4 });
            }
        }

        function createCollisions() {
            this.physics.add.collider(bullets, planes, hitPlane, null, this);
            this.physics.add.collider(bullets, enemyTanks, hitEnemyTank, null, this);
            this.physics.add.collider(bullets, bosses, hitBoss, null, this);
            this.physics.add.collider(tank, planes, takeDamage, null, this);
            this.physics.add.collider(tank, enemyTanks, takeDamage, null, this);
            this.physics.add.collider(tank, bosses, takeDamage, null, this);
        }
    </script>
</body>
</html>
