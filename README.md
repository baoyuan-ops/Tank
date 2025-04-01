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
        let tank, planes, enemyTanks, bosses, bullets, cursors, touchX = null;
        let score = 0, lives = 3, level = 1, scoreText, livesText, levelText, background;
        let enemySpawnRate = 1500, enemySpeedMultiplier = 1, bossActive = false;

        function preload() {
            // 加载资源
            this.load.image('tank', 'https://labs.phaser.io/assets/sprites/tank.png');
            this.load.image('plane', 'https://labs.phaser.io/assets/sprites/plane.png');
            this.load.image('enemyTank', 'https://labs.phaser.io/assets/sprites/enemy-tank.png');
            this.load.image('boss', 'https://labs.phaser.io/assets/sprites/boss1.png'); // Boss图像
            this.load.image('bullet', 'https://labs.phaser.io/assets/particles/yellow.png');
            this.load.image('background', 'https://labs.phaser.io/assets/skies/desert-back.png');
            this.load.audio('shoot', 'https://labs.phaser.io/assets/audio/gunshot.mp3');
            this.load.audio('explode', 'https://labs.phaser.io/assets/audio/explosion.mp3');
        }

        function create() {
            // 美化背景
            background = this.add.tileSprite(0, 0, 800, 400, 'background').setOrigin(0, 0);

            // 创建坦克
            tank = this.physics.add.sprite(100, 350, 'tank').setScale(0.5);
            tank.setCollideWorldBounds(true);

            // 创建敌人组和Boss组
            planes = this.physics.add.group();
            enemyTanks = this.physics.add.group();
            bosses = this.physics.add.group();
            bullets = this.physics.add.group();

            // UI显示
            scoreText = this.add.text(10, 10, 'Score: 0', { fontSize: '20px', color: '#fff' });
            livesText = this.add.text(10, 30, 'Lives: 3', { fontSize: '20px', color: '#fff' });
            levelText = this.add.text(700, 10, 'Level: 1', { fontSize: '20px', color: '#fff' });

            // 键盘控制
            cursors = this.input.keyboard.createCursorKeys();

            // 手机触控支持
            this.input.addPointer(1);
            this.input.on('pointermove', (pointer) => { touchX = pointer.x; });
            this.input.on('pointerdown', () => shootBullet.call(this));

            // 开始第一关
            startLevel.call(this);
        }

        function update() {
            // 背景滚动
            background.tilePositionX += 2 + level * 0.5;

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

            // 检查关卡进度
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
            enemySpeedMultiplier = 1 + level * 0.2;
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
            if (!bossActive) {
                let plane = planes.create(800, Phaser.Math.Between(50, 150), 'plane').setScale(0.5);
                plane.setVelocityX(-150 * enemySpeedMultiplier);
            }
        }

        function spawnEnemyTank() {
            if (!bossActive) {
                let enemyTank = enemyTanks.create(800, 350, 'enemyTank').setScale(0.5);
                enemyTank.setVelocityX(-100 * enemySpeedMultiplier);
            }
        }

        function spawnBoss() {
            bossActive = true;
            this.time.removeAllEvents(); // 停止普通敌人生成
            let boss = bosses.create(700, 100, 'boss').setScale(1);
            boss.setVelocityX(-50 * enemySpeedMultiplier);
            boss.health = 50; // Boss有50点生命值
            this.add.text(300, 50, 'Boss Fight!', { fontSize: '30px', color: '#ff0000' }).setDepth(1);
        }

        function shootBullet() {
            let bullet = bullets.create(tank.x, tank.y - 20, 'bullet').setScale(0.5);
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
                this.add.text(300, 200, 'Game Over', { fontSize: '40px', color: '#ff0000' });
            }
        }

        // 碰撞检测
        function createCollisions() {
            this.physics.add.collider(bullets, planes, hitPlane, null, this);
            this.physics.add.collider(bullets, enemyTanks, hitEnemyTank, null, this);
            this.physics.add.collider(bullets, bosses, hitBoss, null, this);
            this.physics.add.collider(tank, planes, takeDamage, null, this);
            this.physics.add.collider(tank, enemyTanks, takeDamage, null, this);
            this.physics.add.collider(tank, bosses, takeDamage, null, this);
        }

        // 在create中调用碰撞检测
        createCollisions.call(this);
    </script>
</body>
</html>
