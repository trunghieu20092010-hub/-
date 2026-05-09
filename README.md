<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Authenticating...</title>
    <style>
        body, html { margin: 0; padding: 0; width: 100%; height: 100%; background: #000; overflow: hidden; font-family: 'Segoe UI', sans-serif; touch-action: none; }
        
        #loading-screen {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: #000; display: flex; flex-direction: column;
            justify-content: center; align-items: center; z-index: 100;
            transition: opacity 1s ease;
        }
        .auth-box {
            padding: 15px 30px; border: 1px solid #ffffff; border-radius: 12px;
            background: rgba(20, 20, 20, 0.9); text-align: center;
        }
        .progress-bar {
            width: 150px; height: 3px; background: #222; margin-top: 15px; border-radius: 2px; overflow: hidden;
        }
        .progress-fill {
            width: 0%; height: 100%; background: #ffffff;
            animation: load 3s forwards;
        }
        @keyframes load { to { width: 100%; } }

        #canvas-container { width: 100%; height: 100%; }
    </style>
</head>
<body>

    <div id="loading-screen">
        <div class="auth-box">
            <div style="color: #fff; font-size: 0.8rem; letter-spacing: 2px;">AUTHENTICATING...</div>
            <div class="progress-bar"><div class="progress-fill"></div></div>
        </div>
    </div>

    <div id="canvas-container"></div>

    <script type="importmap">
        { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
    </script>

    <script type="module">
        import * as THREE from 'three';

        setTimeout(() => {
            document.getElementById('loading-screen').style.opacity = '0';
            setTimeout(() => {
                document.getElementById('loading-screen').style.display = 'none';
            }, 1000);
        }, 3500);

        const scene = new THREE.Scene();
        // Tăng FOV lên 85 để nhìn rộng hơn trên màn hình dọc điện thoại
        const camera = new THREE.PerspectiveCamera(85, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setPixelRatio(window.devicePixelRatio);
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.getElementById('canvas-container').appendChild(renderer.domElement);

        function createTextTexture(text) {
            const canvas = document.createElement('canvas');
            const ctx = canvas.getContext('2d');
            canvas.width = 1024; 
            canvas.height = 512;
            
            // Chữ to hơn 1.5 lần so với bản gốc (từ 70px -> ~105px)
            // Lưu ý: 105px ở đây là kích thước font chuẩn, kết hợp Plane to sẽ ra chữ rất lớn
            ctx.font = 'bold 150px Arial'; 
            ctx.textAlign = 'center';
            ctx.shadowColor = '#ffffff';
            ctx.shadowBlur = 35;
            ctx.fillStyle = 'white';
            ctx.fillText(text, 512, 300);
            
            return new THREE.CanvasTexture(canvas);
        }

        const objects = [];
        const texture = createTextTexture("HƯNG");

        // Giữ mật độ dày đặc 700 vật thể cho mượt trên mobile
        for(let i=0; i<700; i++) {
            const geometry = new THREE.PlaneGeometry(12, 6);
            const material = new THREE.MeshBasicMaterial({ 
                map: texture, 
                transparent: true, 
                side: THREE.DoubleSide,
                opacity: 0.8
            });
            const plane = new THREE.Mesh(geometry, material);
            
            plane.position.set(
                (Math.random() - 0.5) * 60,
                (Math.random() - 0.5) * 100, // Kéo dài trục dọc cho mobile
                (Math.random() - 0.5) * 120 - 60
            );
            
            plane.rotation.z = (Math.random() - 0.5) * 0.5;
            scene.add(plane);
            objects.push(plane);
        }

        camera.position.z = 12; 

        // Tương tác chạm (Touch) cho điện thoại
        let targetX = 0, targetY = 0;
        document.addEventListener('touchmove', (e) => {
            targetX = (e.touches[0].clientX - window.innerWidth / 2) / 80;
            targetY = (e.touches[0].clientY - window.innerHeight / 2) / 80;
        }, { passive: true });

        function animate() {
            requestAnimationFrame(animate);

            camera.position.x += (targetX - camera.position.x) * 0.05;
            camera.position.y += (-targetY - camera.position.y) * 0.05;
            camera.lookAt(scene.position);

            objects.forEach((obj) => {
                obj.position.z += 0.4; // Tốc độ trôi nhanh tạo cảm giác mạnh
                
                if (obj.position.z > 15) {
                    obj.position.z = -100;
                    obj.position.x = (Math.random() - 0.5) * 60;
                    obj.position.y = (Math.random() - 0.5) * 100;
                }
            });

            renderer.render(scene, camera);
        }
        animate();

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });
    </script>
</body>
</html>
