# Tank
a game about Tank
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
        let tank, planes, bullets, cursors, touchX = null;

        function preload() {
            // 使用占位符图片，实际开发时替换为坦克、飞机、子弹的图像
            this.load.image('tank', 'https://via.placeholder.com/50x30.png?text=Tank');
            this.load.image('plane', 'https://via.placeholder.com/40x20.png?text=Plane');
            this.load.image('bullet', 'https://via.placeholder.com/5x15.png?text=Bullet');
        }

        function create() {
            // 创建坦克
            tank = this.physics.add.sprite(100, 350, 'tank');
            tank.setCollideWorldBounds(true);

            // 创建飞机和子弹组
            planes = this.physics.add.group();
            bullets = this.physics.add.group();

            // 键盘控制
            cursors = this.input.keyboard.createCursorKeys();

            // 手机触控支持
            this.input.addPointer(1);
            this.input.on('pointermove', (pointer) => {
                touchX = pointer.x;
            });
            this.input.on('pointerdown', () => shootBullet.call(this));

            // 每隔1.5秒生成飞机
            this.time.addEvent({
                delay: 1500,
                callback: spawnPlane,
                callbackScope: this,
                loop: true
            });

            // 碰撞检测
            this.physics.add.collider(bullets, planes, hitPlane, null, this);
        }

        function update() {
            // 键盘控制坦克移动
            if (cursors.left.isDown) {
                tank.setVelocityX(-200);
            } else if (cursors.right.isDown) {
                tank.setVelocityX(200);
            } else if (touchX !== null) {
                // 手机触控移动
                if (touchX < tank.x) tank.setVelocityX(-200);
                else if (touchX > tank.x) tank.setVelocityX(200);
                else tank.setVelocityX(0);
            } else {
                tank.setVelocityX(0);
            }

            // 空格键射击
            if (Phaser.Input.Keyboard.JustDown(cursors.space)) {
                shootBullet.call(this);
            }
        }

        function spawnPlane() {
            let plane = planes.create(800, Phaser.Math.Between(50, 150), 'plane');
            plane.setVelocityX(-150); // 飞机向左飞
        }

        function shootBullet() {
            let bullet = bullets.create(tank.x, tank.y - 20, 'bullet');
            bullet.setVelocityY(-400); // 子弹向上发射
            bullet.checkWorldBounds = true;
            bullet.outOfBoundsKill = true; // 出界销毁
        }

        function hitPlane(bullet, plane) {
            bullet.destroy();
            plane.destroy();
        }
    </script>
</body>
</html>
