# Virtual Touch

Sistema experimental de interacao humano-computador usando webcam, visao computacional e gestos das maos. O projeto foi desenhado para evoluir de um preview seguro para diferentes modos de controle, sem acoplar a deteccao de maos ao Windows ou a um dispositivo de entrada especifico.

## Conceito

```text
webcam -> frames -> visao computacional -> landmarks -> gestos
	-> eventos -> controladores de input -> aplicacao/ sistema operacional
```

O MediaPipe fica isolado no adaptador `vision.hand_tracker`. O restante da aplicacao trabalha com modelos internos de landmarks, eventos e coordenadas.

## Implementado

- Captura de webcam com OpenCV e espelhamento configuravel.
- Deteccao de uma ou duas maos com MediaPipe em modo video.
- Deteccao simultanea de ate duas maos, identificadas como `LEFT HAND` e `RIGHT HAND`.
- Conversao dos landmarks para modelos internos do projeto.
- Identificacao independente das pontas do polegar, indicador, medio, anelar e minimo.
- Preview com landmarks, indicador, gesto atual, coordenadas e FPS.
- Cena 2D com ponto, linha e retangulo selecionaveis pelo dedo indicador.
- Movimentacao do objeto selecionado dentro da area virtual.
- Dados de distancia e angulo entre as duas maos para futuras transformacoes.
- Velocidade, direcao, suavizacao leve e previsao curta por dedo.
- Pinça com thresholds de ativacao/liberacao e recuperacao de perda momentanea.
- Ponteiro virtual semitransparente persistente durante pequenas perdas de tracking.
- Filtro adaptativo com dead zone, aceleracao por velocidade e modos `absolute`/`relative`.
- Margens, offsets, sensibilidade e velocidade do ponteiro configuraveis.
- Backend Windows via `user32` para mover, clicar, arrastar e rolar.
- Toggle de camera por botao no canvas ou `Ctrl+Shift+C`.
- Ativacao explicita do controle por botao `CONTROL ON`.
- Dois modos de uso: `VIRTUAL` mantém a cena interna; `COMPUTER` envia input real ao Windows.
- Emergencia por `ESC`, liberando botoes e desativando o controle.
- Barramento interno de eventos.
- Configuracao tipada e area de interacao.
- Contratos para mouse, teclado, touch e janelas.
- Modo demonstracao seguro: controle real inicia desativado.
- Testes unitarios para eventos, coordenadas, landmarks e protecao do input.

## Ainda planejado

- Clique direito/middle e scroll dependem de calibracao dos gestos secundarios.
- Escala e rotacao de objetos usando a distancia e a posicao relativa das duas maos.
- Calibracao interativa, suavizacao aplicada e suporte a multiplos monitores.
- Backends reais de teclado, touch e adaptadores de aplicacoes.
- Interface de configuracao, overlay avancado e modo 3D.
- Screenshots, GIF de demonstracao e documentacao de publicacao.

## Arquitetura

```text
src/core/          ciclo de vida, estado e eventos
src/vision/        webcam, processamento e landmarks
src/gestures/      detector e motor de gestos
src/input/         contratos de mouse, teclado, touch e janelas
src/interaction/   calibracao e coordenadas
src/ui/            preview, overlay e configuracoes
src/config/        valores padrao e Settings
```

## Tecnologias

- Python 3.11+
- OpenCV
- MediaPipe Tasks e o modelo local `assets/models/hand_landmarker.task`
- NumPy
- pytest e Ruff para desenvolvimento

## Instalacao

No PowerShell, a partir de `virtual-touch/`:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

O modelo de hand tracking ja esta incluido em `assets/models/hand_landmarker.task`. Se ele for removido, a aplicacao informara o caminho esperado e nao iniciara parcialmente.

## Execucao

Com uma webcam conectada:

```powershell
python -m src.main
```

Pressione `Q` para encerrar. `ESC` e a parada de emergencia: desativa o controle e libera botoes sem fechar a aplicacao.

O controle real inicia desligado no modo `VIRTUAL`. Clique em `CONTROL OFF` ou use `Ctrl+Shift+M` para alternar para `COMPUTER` e ativar o input do Windows. O botao `CAMERA ON/OFF` e o atalho global `Ctrl+Shift+C` usam o mesmo estado central; ao desligar a camera, o tracker e liberado, mas o canvas, objetos e configuracoes permanecem.

Com o controle ativado, o ponteiro virtual movimenta o cursor real para hover. A pinça gera `MOUSE DOWN`/`MOUSE UP`, permitindo clique e arraste; dois dedos geram scroll proporcional a velocidade.

## Testes desta etapa

1. **Velocidade:** mova rapidamente o indicador em zigue-zague e observe `V` e `D` no debug; reduza `smoothing` para `0.0` em `Settings` para resposta imediata.
2. **Duas maos:** coloque as duas maos na camera e confirme `LEFT HAND`, `RIGHT HAND`, dois indicadores e a linha entre eles.
3. **Pinça:** aproxime polegar e indicador de uma mao; o debug deve mostrar `G:PINCH`. Separe-os para liberar.
4. **Escala:** faça pinça com as duas maos sobre o mesmo objeto; afaste os indicadores para aumentar e aproxime para diminuir.
5. **Movimento rápido:** mantendo a pinça, mova uma mao rapidamente; a previsao curta acompanha o movimento sem ativar o mouse real.
6. **Recuperação:** fixe um objeto com as duas maos, retire uma delas por alguns frames e confirme que a outra continua detectada; ao retornar, a mao pode iniciar nova fixacao.

Os parametros `pointer_mode`, `pointer_dead_zone`, `pointer_acceleration`, `pointer_speed`, `pointer_screen_margin`, `pointer_offset_x`, `pointer_offset_y`, `smoothing`, `prediction_seconds`, `pinch_threshold`, `pinch_release_threshold`, `max_missing_frames`, `target_fps` e `max_hands` ficam em `src/config/settings.py`.

## Gestos de input

O mapa fica em `src/input/gesture_mapping.py`: `PINCH` controla clique esquerdo/arraste, `INDEX_MIDDLE_PINCH` clique direito, `INDEX_RING_PINCH` clique do meio, `INDEX_PINKY_PINCH` duplo clique e `TWO_FINGERS` scroll. O backend Windows fica em `src/input/windows_mouse_controller.py`; a visao e o Gesture Engine nao chamam a API do Windows diretamente.

## Testes

```powershell
pytest
```

## Estrutura de diretorios

```text
src/              codigo da aplicacao
tests/            testes por camada
assets/           icones, imagens e modelo local de hand tracking
config/           configuracoes externas futuras
docs/             arquitetura, desenvolvimento e roadmap
```

## Seguranca

O estado inicial usa `camera_enabled=True`, `demo_mode=True` e `control_enabled=False`. O `InputManager` ignora eventos de sistema enquanto essas protecoes estiverem ativas. O backend real rastreia botoes pressionados e `ESC` chama `release_all()` antes de retornar ao modo seguro.

## Licenca

Este projeto esta disponivel sob a licenca MIT. Consulte [LICENSE](LICENSE).
