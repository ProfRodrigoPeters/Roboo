<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Protótipo de Robô Giratório</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes spin {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }
        @keyframes pulse-admiration {
            0% { box-shadow: 0 0 5px #00ff00; }
            50% { box-shadow: 0 0 20px #00ff00, 0 0 30px #00ff00; }
            100% { box-shadow: 0 0 5px #00ff00; }
        }
        @keyframes pulse-caution {
            0% { box-shadow: 0 0 5px #ff0000; }
            50% { box-shadow: 0 0 20px #ff0000, 0 0 30px #ff0000; }
            100% { box-shadow: 0 0 5px #ff0000; }
        }
        .radian-mark {
            position: absolute;
            width: 2px;
            height: 20px;
            background: #000;
            transform-origin: bottom center;
        }
        .robot-arm {
            transition: all 0.5s ease;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center justify-center p-4">
    <div class="max-w-4xl w-full bg-white rounded-xl shadow-2xl p-6">
        <h1 class="text-3xl font-bold text-center mb-6">Protótipo de Plataforma Robótica Giratória</h1>
        
        <!-- Controles -->
        <div class="flex flex-wrap gap-4 justify-center mb-8">
            <button id="spin-platform" class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition">
                Girar Plataforma
            </button>
            <button id="admiration-mode" class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition">
                Modo Admiração
            </button>
            <button id="caution-mode" class="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition">
                Modo Cautela
            </button>
            <button id="stop-all" class="px-4 py-2 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition">
                Parar Tudo
            </button>
        </div>

        <!-- Plataforma -->
        <div class="relative mx-auto mb-8" style="width: 600px; height: 600px;">
            <!-- Base Quadrada -->
            <div class="absolute inset-0 bg-amber-100 border-4 border-amber-600 rounded-lg flex items-center justify-center">
                <div id="radian-marks" class="relative w-full h-full"></div>
                <div class="text-amber-800 font-bold text-lg">BASE QUADRADA (60cm)</div>
            </div>
            
            <!-- Base Redonda Giratória -->
            <div id="rotating-platform" class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 
                w-96 h-96 bg-blue-100 rounded-full border-4 border-blue-600 flex items-center justify-center transition-transform duration-1000">
                <div class="text-blue-800 font-bold text-lg">BASE REDONDA (45cm)</div>
                <div class="absolute w-full h-1 bg-blue-600 top-1/2 left-0"></div>
                
                <!-- Robô -->
                <div id="robot" class="absolute bottom-1/2 left-1/2 transform -translate-x-1/2 cursor-pointer"
                    style="width: 120px;">
                    <div class="robot-body bg-gray-400 w-24 h-32 mx-auto rounded-t-lg relative">
                        <div class="robot-head bg-gray-300 w-16 h-12 mx-auto rounded-t-full absolute -top-4 left-1/2 transform -translate-x-1/2 flex justify-center items-center">
                            <div class="robot-eye left-eye bg-white w-4 h-4 rounded-full absolute left-2"></div>
                            <div class="robot-eye right-eye bg-white w-4 h-4 rounded-full absolute right-2"></div>
                        </div>
                        <div class="robot-arm left-arm absolute top-8 left-0 bg-gray-500 w-4 h-16 rounded"></div>
                        <div class="robot-arm right-arm absolute top-8 right-0 bg-gray-500 w-4 h-16 rounded"></div>
                        <div class="robot-base bg-gray-600 w-28 h-6 mx-auto rounded absolute bottom-0"></div>
                        <div id="robot-indicator" class="absolute bottom-2 left-1/2 transform -translate-x-1/2 w-6 h-6 rounded-full"></div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Status -->
        <div class="text-center">
            <div id="status" class="text-xl font-semibold mb-2">Estado: Pronto</div>
            <div id="angle-display" class="text-lg">Ângulo atual: 0 radianos</div>
        </div>
    </div>

    <script>
        // Inicialização
        document.addEventListener('DOMContentLoaded', function() {
            // Adicionar marcas de radianos
            const radianContainer = document.getElementById('radian-marks');
            const angles = [0, Math.PI/4, Math.PI/2, 3*Math.PI/4, Math.PI, 5*Math.PI/4, 3*Math.PI/2, 7*Math.PI/4];
            
            angles.forEach((angle, index) => {
                const mark = document.createElement('div');
                mark.className = 'radian-mark';
                mark.style.bottom = '50%';
                mark.style.left = '50%';
                mark.style.transform = `rotate(${angle}rad) translateY(-150px)`;
                
                const label = document.createElement('div');
                label.textContent = `${angle}π`;
                label.style.position = 'absolute';
                label.style.transform = 'translateX(-50%)';
                label.style.bottom = '-30px';
                
                mark.appendChild(label);
                radianContainer.appendChild(mark);
            });

            // Variáveis de estado
            let isSpinning = false;
            let currentMode = 'neutral';
            let currentAngle = 0;
            const rotationSpeed = 1; // graus por frame

            // Elementos
            const platform = document.getElementById('rotating-platform');
            const robot = document.getElementById('robot');
            const robotIndicator = document.getElementById('robot-indicator');
            const statusDisplay = document.getElementById('status');
            const angleDisplay = document.getElementById('angle-display');
            const spinButton = document.getElementById('spin-platform');
            const admirationButton = document.getElementById('admiration-mode');
            const cautionButton = document.getElementById('caution-mode');
            const stopButton = document.getElementById('stop-all');

            // Controles
            spinButton.addEventListener('click', function() {
                isSpinning = !isSpinning;
                spinButton.textContent = isSpinning ? 'Parar Giro' : 'Girar Plataforma';
                statusDisplay.textContent = isSpinning ? 'Estado: Girando' : 'Estado: Parado';
            });

            admirationButton.addEventListener('click', function() {
                currentMode = 'admiration';
                updateRobotAppearance();
            });

            cautionButton.addEventListener('click', function() {
                currentMode = 'caution';
                updateRobotAppearance();
            });

            stopButton.addEventListener('click', function() {
                isSpinning = false;
                currentMode = 'neutral';
                spinButton.textContent = 'Girar Plataforma';
                statusDisplay.textContent = 'Estado: Parado';
                updateRobotAppearance();
            });

            // Atualizar aparência do robô
            function updateRobotAppearance() {
                // Limpar todas as classes de animação primeiro
                robotIndicator.className = 'absolute bottom-2 left-1/2 transform -translate-x-1/2 w-6 h-6 rounded-full';
                
                if (currentMode === 'admiration') {
                    robotIndicator.classList.add('bg-green-500');
                    robotIndicator.style.animation = 'pulse-admiration 2s infinite';
                    document.querySelector('.robot-body').style.backgroundColor = '#4fd1c5';
                    document.querySelectorAll('.robot-eye').forEach(eye => eye.style.backgroundColor = '#00ff88');
                    document.querySelector('.robot-head').style.backgroundColor = '#38b2ac';
                    statusDisplay.textContent = 'Estado: Modo Admiração';
                } 
                else if (currentMode === 'caution') {
                    robotIndicator.classList.add('bg-red-500');
                    robotIndicator.style.animation = 'pulse-caution 2s infinite';
                    document.querySelector('.robot-body').style.backgroundColor = '#f56565';
                    document.querySelectorAll('.robot-eye').forEach(eye => eye.style.backgroundColor = '#ff0000');
                    document.querySelector('.robot-head').style.backgroundColor = '#e53e3e';
                    statusDisplay.textContent = 'Estado: Modo Cautela';
                } 
                else {
                    robotIndicator.classList.add('bg-gray-500');
                    document.querySelector('.robot-body').style.backgroundColor = '#9ca3af';
                    document.querySelectorAll('.robot-eye').forEach(eye => eye.style.backgroundColor = '#ffffff');
                    document.querySelector('.robot-head').style.backgroundColor = '#d1d5db';
                    statusDisplay.textContent = isSpinning ? 'Estado: Girando' : 'Estado: Parado';
                }
            }

            // Animação de rotação
            function animate() {
                if (isSpinning) {
                    currentAngle += rotationSpeed;
                    if (currentAngle >= 360) currentAngle = 0;
                    
                    platform.style.transform = `translate(-50%, -50%) rotate(${currentAngle}deg)`;
                    
                    // Atualizar exibição do ângulo em radianos
                    const radians = (currentAngle * Math.PI / 180).toFixed(2);
                    angleDisplay.textContent = `Ângulo atual: ${radians} radianos`;
                    
                    // Movimento do robô (opcional)
                    const armMovement = Math.sin(currentAngle * Math.PI / 180) * 15;
                    document.querySelector('.robot-arm.left-arm').style.height = `${60 + armMovement}px`;
                    document.querySelector('.robot-arm.right-arm').style.height = `${60 - armMovement}px`;
                }
                
                requestAnimationFrame(animate);
            }

            // Interatividade com o robô
            robot.addEventListener('click', function() {
                alert('O robô foi ativado! Esta é uma demonstração de como ele poderia responder a interações.');
            });

            // Iniciar animação
            animate();
            updateRobotAppearance();
        });
    </script>
</body>
</html>
# Roboo
