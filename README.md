<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <!-- Viewport otimizada para evitar gestos e zoom indesejados no iOS -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Jogo de Coleta Mobile & Safari</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
        }

        body {
            margin: 0;
            padding: 10px;
            background-color: #1a1a1a;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: sans-serif;
            overflow: hidden; /* Evita rolagem da página */
        }

        canvas {
            border: 3px solid #333;
            max-width: 100%;
            max-height: 55vh;
            background-color: black;
            /* Compatibilidade com gestos no Safari/iOS */
            touch-action: none;
            -webkit-touch-callout: none;
        }

        /* Controles Virtuais (D-Pad) */
        .controles {
            display: grid;
            grid-template-columns: repeat(3, 65px);
            grid-template-rows: repeat(2, 65px);
            gap: 10px;
            margin-top: 15px;
        }

        .btn {
            background: #333;
            color: white;
            border: 2px solid #555;
            border-radius: 12px;
            font-size: 24px;
            display: flex;
            justify-content: center;
            align-items: center;
            /* Trava zoom e gestos nos botões */
            touch-action: none;
            -webkit-touch-callout: none;
            cursor: pointer;
        }

        .btn:active {
            background: #555;
        }

        #btn-cima { grid-column: 2; grid-row: 1; }
        #btn-esquerda { grid-column: 1; grid-row: 2; }
        #btn-baixo { grid-column: 2; grid-row: 2; }
        #btn-direita { grid-column: 3; grid-row: 2; }
    </style>
</head>
<body>
    <canvas id="tela" width="600" height="600"></canvas>

    <div class="controles">
        <button class="btn" id="btn-cima">▲</button>
        <button class="btn" id="btn-esquerda">◄</button>
        <button class="btn" id="btn-baixo">▼</button>
        <button class="btn" id="btn-direita">►</button>
    </div>

    <script>
        const canvas = document.getElementById("tela");
        const ctx = canvas.getContext("2d");

        // Estado do Jogador
        let x = 50;
        let y = 50;
        const tamanho = 30;
        const velocidade = 5;

        // Estado da Fruta
        const tamanhoFruta = 20;
        let frutaX = 0;
        let frutaY = 0;
        let pontos = 0;

        // Mapeamento de Entradas
        const teclas = {};

        // 1. Suporte a Teclado (PC)
        document.addEventListener("keydown", (e) => teclas[e.key] = true);
        document.addEventListener("keyup", (e) => teclas[e.key] = false);

        // 2. Suporte Avançado a Touch e Mouse (Safari iOS & Mobile)
        function vincularBotaoControle(id, tecla) {
            const btn = document.getElementById(id);

            const ativar = (e) => {
                if (e.cancelable) e.preventDefault();
                teclas[tecla] = true;
            };

            const desativar = (e) => {
                if (e.cancelable) e.preventDefault();
                teclas[tecla] = false;
            };

            // Eventos de Toque (Mobile/Safari)
            btn.addEventListener("touchstart", ativar, { passive: false });
            btn.addEventListener("touchend", desativar, { passive: false });
            btn.addEventListener("touchcancel", desativar, { passive: false });

            // Eventos de Mouse (Fallback Desktop)
            btn.addEventListener("mousedown", ativar);
            btn.addEventListener("mouseup", desativar);
            btn.addEventListener("mouseleave", desativar);
        }

        vincularBotaoControle("btn-cima", "ArrowUp");
        vincularBotaoControle("btn-baixo", "ArrowDown");
        vincularBotaoControle("btn-esquerda", "ArrowLeft");
        vincularBotaoControle("btn-direita", "ArrowRight");

        function gerarFruta() {
            // Garante dimensões válidas antes de posicionar a fruta
            const larguraMaxima = (canvas.width || 600) - tamanhoFruta;
            const alturaMaxima = (canvas.height || 600) - tamanhoFruta;
            
            frutaX = Math.floor(Math.random() * larguraMaxima);
            frutaY = Math.floor(Math.random() * alturaMaxima);
        }

        function atualizar() {
            // Movimento
            if (teclas["ArrowRight"]) x += velocidade;
            if (teclas["ArrowLeft"])  x -= velocidade;
            if (teclas["ArrowUp"])    y -= velocidade;
            if (teclas["ArrowDown"])  y += velocidade;

            // Limites do Canvas (Paredes)
            if (x < 0) x = 0;
            if (y < 0) y = 0;
            if (x + tamanho > canvas.width) x = canvas.width - tamanho;
            if (y + tamanho > canvas.height) y = canvas.height - tamanho;

            // Colisão com a Fruta
            if (
                x < frutaX + tamanhoFruta &&
                x + tamanho > frutaX &&
                y < frutaY + tamanhoFruta &&
                y + tamanho > frutaY
            ) {
                pontos += 1;
                gerarFruta();
            }
        }

        function desenhar() {
            // Limpa a tela
            ctx.fillStyle = "black";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Desenha a Fruta
            ctx.fillStyle = "red";
            ctx.fillRect(frutaX, frutaY, tamanhoFruta, tamanhoFruta);

            // Desenha o Jogador
            ctx.fillStyle = "green";
            ctx.fillRect(x, y, tamanho, tamanho);

            // Desenha a Pontuação
            ctx.fillStyle = "white";
            ctx.font = "20px Arial";
            ctx.fillText(`Pontos: ${pontos}`, 20, 35);
        }

        function loop() {
            atualizar();
            desenhar();
            requestAnimationFrame(loop);
        }

        // Inicialização segura após o carregamento da página
        window.addEventListener("load", () => {
            gerarFruta();
            loop();
        });
    </script>
</body>
</html>
