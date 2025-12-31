# nb
可爱的
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>烟花贺新春 - 新年成语祝福</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: "Microsoft YaHei", sans-serif;
            background: #000;
            overflow: hidden;
            color: #fff;
            text-align: center;
        }
        #canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 1;
        }
        .control-panel {
            position: relative;
            z-index: 100;
            background: rgba(0, 0, 0, 0.7);
            border-radius: 15px;
            padding: 20px;
            margin: 20px auto;
            max-width: 400px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 215, 0, 0.3);
        }
        h1 {
            color: #FFD700;
            margin-bottom: 10px;
            text-shadow: 0 0 10px #FF4500;
        }
        .subtitle {
            color: #FFA07A;
            margin-bottom: 20px;
            font-size: 0.9em;
        }
        button {
            background: linear-gradient(45deg, #FF4500, #FFD700);
            border: none;
            color: #000;
            padding: 12px 20px;
            margin: 8px;
            border-radius: 50px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s;
            font-size: 1em;
        }
        button:hover {
            transform: scale(1.05);
            box-shadow: 0 0 15px #FFD700;
        }
        .instructions {
            margin-top: 20px;
            font-size: 0.85em;
            color: #aaa;
            line-height: 1.5;
        }
        .highlight {
            color: #FFD700;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <!-- 烟花画布 -->
    <canvas id="canvas"></canvas>
    
    <!-- 控制面板 -->
    <div class="control-panel">
        <h1>🎆 烟花贺新春 🎇</h1>
        <p class="subtitle">点击页面任意处发射烟花 · 点击飘落的文字可自定义祝福</p>
        <div>
            <button onclick="toggleFireworks()">暂停/继续烟花</button>
            <button onclick="addMore()">多发一些</button>
            <button onclick="changeWords()">换一组成语</button>
        </div>
        <p class="instructions">
            <span class="highlight">操作提示：</span><br>
            1. 点击页面任意空白处可发射一枚烟花<br>
            2. 烟花爆炸后出现的<span class="highlight">成语可以点击编辑</span><br>
            3. 成语列表当前为：<span id="currentWords">万事如意, 恭喜发财, 新年快乐, 龙马精神, 阖家幸福, 财源广进, 心想事成, 吉星高照</span>
        </p>
    </div>

    <script>
        // ========== 核心配置 ==========
        // 在这里修改默认显示的成语，可以任意增减或替换
        let wordList = [
            "万事如意", "恭喜发财", "新年快乐", "龙马精神",
            "阖家幸福", "财源广进", "心想事成", "吉星高照",
            "步步高升", "福星高照", "年年有余", "大吉大利"
        ];
        let autoFireInterval = 800; // 自动发射间隔(毫秒)
        let wordAppearChance = 0.7; // 文字出现概率 (0-1)

        // ========== 变量初始化 ==========
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        let fireworks = [], particles = [], texts = [];
        let isAnimating = true, autoFireTimer;
        let width = window.innerWidth, height = window.innerHeight;
        canvas.width = width; canvas.height = height;

        // ========== 核心函数：创建烟花 ==========
        function createFirework(x, y, targetX, targetY) {
            const firework = {
                x: x || Math.random() * width,
                y: y || height,
                targetX: targetX || Math.random() * width,
                targetY: targetY || Math.random() * (height * 0.4) + 100,
                speed: 2 + Math.random() * 3,
                brightness: 200 + Math.random() * 55,
                color: `hsl(${Math.random() * 360}, 100%, 60%)`,
                trail: [],
                exploded: false
            };
            fireworks.push(firework);
        }

        // ========== 核心函数：烟花爆炸 ==========
        function explode(firework) {
            const particleCount = 60 + Math.random() * 80;
            const power = 6 + Math.random() * 7;

            // 创建爆炸粒子
            for (let i = 0; i < particleCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const velocity = Math.random() * power;
                particles.push({
                    x: firework.x, y: firework.y,
                    vx: Math.cos(angle) * velocity,
                    vy: Math.sin(angle) * velocity,
                    life: 100,
                    decay: 0.4 + Math.random() * 1.2,
                    color: firework.color,
                    size: 1 + Math.random() * 3
                });
            }

            // 随机决定是否显示一个文字成语
            if (Math.random() < wordAppearChance && wordList.length > 0) {
                const selectedWord = wordList[Math.floor(Math.random() * wordList.length)];
                texts.push({
                    x: firework.x, y: firework.y,
                    content: selectedWord,
                    size: 20 + Math.random() * 15,
                    color: `hsl(${Math.random() * 360}, 100%, 70%)`,
                    vx: (Math.random() - 0.5) * 2,
                    vy: -2 - Math.random() * 2,
                    life: 300, // 文字显示时间
                    decay: 0.7
                });
            }
        }

        // ========== 核心函数：绘制与动画循环 ==========
        function draw() {
            // 半透明黑色背景，制造拖尾效果
            ctx.fillStyle = 'rgba(0, 0, 0, 0.15)';
            ctx.fillRect(0, 0, width, height);

            // 1. 更新并绘制上升中的烟花
            fireworks.forEach((fw, idx) => {
                const dx = fw.targetX - fw.x;
                const dy = fw.targetY - fw.y;
                const dist = Math.sqrt(dx*dx + dy*dy);
                
                fw.x += (dx / dist) * fw.speed;
                fw.y += (dy / dist) * fw.speed;
                fw.trail.push({x: fw.x, y: fw.y});
                if (fw.trail.length > 8) fw.trail.shift();

                // 绘制尾迹
                ctx.beginPath();
                ctx.moveTo(fw.trail[0].x, fw.trail[0].y);
                for (let i = 1; i < fw.trail.length; i++) {
                    ctx.lineTo(fw.trail[i].x, fw.trail[i].y);
                }
                ctx.strokeStyle = fw.color;
                ctx.lineWidth = 2;
                ctx.stroke();

                // 绘制烟花头
                ctx.beginPath();
                ctx.arc(fw.x, fw.y, 3, 0, Math.PI * 2);
                ctx.fillStyle = fw.color;
                ctx.fill();

                // 到达目标点则爆炸
                if (dist < 5) {
                    fw.exploded = true;
                    explode(fw);
                    fireworks.splice(idx, 1);
                }
            });

            // 2. 更新并绘制爆炸粒子
            particles.forEach((p, idx) => {
                p.x += p.vx;
                p.y += p.vy;
                p.vy += 0.05; // 重力
                p.life -= p.decay;

                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size * (p.life/100), 0, Math.PI * 2);
                ctx.fillStyle = p.color;
                ctx.fill();

                if (p.life <= 0) particles.splice(idx, 1);
            });

            // 3. 更新并绘制文字
            texts.forEach((text, idx) => {
                text.x += text.vx;
                text.y += text.vy;
                text.life -= text.decay;

                ctx.font = `bold ${text.size}px "Microsoft YaHei", sans-serif`;
                ctx.fillStyle = text.color.replace(')', `, ${text.life/300})`).replace('hsl', 'hsla');
                ctx.textAlign = 'center';
                ctx.fillText(text.content, text.x, text.y);

                if (text.life <= 0) texts.splice(idx, 1);
            });

            // 4. 继续动画循环
            if (isAnimating) requestAnimationFrame(draw);
        }

        // ========== 交互控制函数 ==========
        function toggleFireworks() {
            isAnimating = !isAnimating;
            if (isAnimating) {
                draw();
                startAutoFire();
            } else {
                clearInterval(autoFireTimer);
            }
        }

        function addMore() {
            for (let i = 0; i < 5; i++) {
                setTimeout(() => createFirework(), i * 150);
            }
        }

        function changeWords() {
            // 这里可以预设多组不同的成语，每次点击随机切换一组
            const wordGroups = [
                ["一帆风顺", "二龙腾飞", "三羊开泰", "四季平安", "五福临门", "六六大顺", "七星高照", "八方来财"],
                ["鸿运当头", "喜气洋洋", "金玉满堂", "福寿安康", "笑口常开", "天赐洪福", "好运连连", "欢天喜地"],
                ["学业进步", "事业有成", "身体健康",  "阖家团圆", "蒸蒸日上", "鹏程万里", "前程似锦", "马到成功"]
            ];
            wordList = wordGroups[Math.floor(Math.random() * wordGroups.length)];
            document.getElementById('currentWords').textContent = wordList.join(', ');
            
            // 视觉反馈
            const btn = event.target;
            btn.textContent = '已更换！';
            btn.style.background = 'linear-gradient(45deg, #00FF00, #00CED1)';
            setTimeout(() => {
                btn.textContent = '换一组成语';
                btn.style.background = 'linear-gradient(45deg, #FF4500, #FFD700)';
            }, 800);
        }

        function startAutoFire() {
            clearInterval(autoFireTimer);
            autoFireTimer = setInterval(() => {
                if (fireworks.length < 15 && Math.random() > 0.3) {
                    createFirework();
                }
            }, autoFireInterval);
        }

        // ========== 事件监听与初始化 ==========
        // 点击页面发射烟花
        canvas.addEventListener('click', (e) => {
            createFirework(e.clientX, height, e.clientX, e.clientY - 50);
        });

        // 点击文字进行编辑
        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;

            texts.forEach((text, idx) => {
                ctx.font = `bold ${text.size}px "Microsoft YaHei", sans-serif`;
                const metrics = ctx.measureText(text.content);
                const textWidth = metrics.width;
                const textHeight = text.size;

                if (x > text.x - textWidth/2 && x < text.x + textWidth/2 &&
                    y > text.y - textHeight && y < text.y) {
                    const newWord = prompt('编辑祝福语，请输入新的成语或祝福：', text.content);
                    if (newWord && newWord.trim() !== '') {
                        text.content = newWord.trim();
                        // 同时将这个新词加入随机词库，以便后续使用
                        if (!wordList.includes(newWord.trim())) {
                            wordList.push(newWord.trim());
                            document.getElementById('currentWords').textContent = wordList.join(', ');
                        }
                    }
                }
            });
        });

        // 窗口大小调整
        window.addEventListener('resize', () => {
            width = canvas.width = window.innerWidth;
            height = canvas.height = window.innerHeight;
        });

        // ========== 启动 ==========
        // 初始发射一波烟花
        for (let i = 0; i < 3; i++) {
            setTimeout(() => createFirework(), i * 400);
        }
        // 开始动画循环和自动发射
        draw();
        startAutoFire();
    </script>
</body>
</html>
