<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tank Sky Defender</title>
    <script src="https://cdn.jsdelivr.net/npm/phaser@3.55.2/dist/phaser.min.js"></script>
    <style>
        body { margin: 0; padding: 0; display: flex; justify-content: center; align-items: center; height: 100vh; background: #87CEEB; }
        canvas { max-width: 100%; max-height: 100%; }
    </style>
</head>
<body>
    <script>
        const config = {
            type: Phaser.AUTO,
            width: 800,
            height: 400,
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

        let game = new Phaser.Game(config);
        let tank, planes, enemyTanks, bullets, cursors, touchX = null;
        let score = 0, lives = 3, scoreText, livesText, background;

        function preload() {
            // 加载占位符图片和音效
            this.load.image('tank', 'https://via.placeholder.com/50x30.png?text=Tank');
            this.load.image('plane', 'https://via.placeholder.com/40x20.png?text=Plane');
            this.load.image('enemyTank', 'https://via.placeholder.com/50x30.png?text=EnemyTank');
            this.load.image('bullet', 'https://via.placeholder.com/5x15.png?text=Bullet');
            this.load.image('background', 'https://via.placeholder.com/1600x400.png?text=Background');
            this.load.audio('shoot', 'https://labs.phaser.io/assets/audio/gunshot.mp3');
            this.load.audio('explode', 'https://labs.phaser.io/assets/audio/explosion.mp3');
        }

        function create() {
            // 滚动背景
            background = this.add.tileSprite(0, 0, 800, 400, 'background').setOrigin(0, 0);

            // 创建坦克
            tank = this.physics.add.sprite(100, 350, 'tank');
            tank.setCollideWorldBounds(true);

            // 创建敌人组
            planes = this.physics.add.group();
            enemyTanks = this.physics.add.group();
            bullets = this.physics.add.group();

            // 得分和生命值显示
            scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '20px', color: '#fff' });
            livesText = this.add.text(10, 30, 'Lives: 3', { fontSize: '20px', color: '#fff' });

            // 键盘控制
            cursors = this.input.keyboard.createCursorKeys();

            // 手机触控支持
            this.input.addPointer(1);
            this.input.on('pointermove', (pointer) => { touchX = pointer.x; });
            this.input.on('pointerdown', () => shootBullet.call(this));

            // 定时生成敌人
            this.time.addEvent({ delay: 1500, callback: spawnPlane, callbackScope: this, loop: true });
            this.time.addEvent({ delay: 3000, callback: spawnEnemyTank, callbackScope: this, loop: true });

            // 碰撞检测
            this.physics.add.collider(bullets, planes, hitPlane, null, this);
            this.physics.add.collider(bullets, enemyTanks, hitEnemyTank, null, this);
            this.physics.add.collider(tank, planes, takeDamage, null, this);
            this.physics.add.collider(tank, enemyTanks, takeDamage, null, this);
        }

        function update() {
            // 背景滚动
            background.tilePositionX += 2;

            // 坦克移动
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

            // 射击
            if (Phaser.Input.Keyboard.JustDown(cursors.space)) {
                shootBullet.call(this);
            }
        }

        function spawnPlane() {
            let plane = planes.create(800, Phaser.Math.Between(50, 150), 'plane');
            plane.setVelocityX(-150);
        }

        function spawnEnemyTank() {
            let enemyTank = enemyTanks.create(800, 350, 'enemyTank');
            enemyTank.setVelocityX(-100);
        }

        function shootBullet() {
            let bullet = bullets.create(tank.x, tank.y - 20, 'bullet');
            bullet.setVelocityY(-400);
            bullet.checkWorldBounds = true;
            bullet.outOfBoundsKill = true;
            this.sound.play('shoot');
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

        function takeDamage(tank, enemy) {
            enemy.destroy();
            lives -= 1;
            livesText.setText('Lives: ' + lives);
            this.sound.play('explode');
            if (lives <= 0) {
                this.physics.pause();
                this.add.text(300, 200, 'Game Over', { fontSize: '40px', color: '#ff0000' });
            }
        }
    </script>
</body>
</html>
