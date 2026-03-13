# Arduinopy Hand Recognition

Projeto de visão computacional para reconhecimento de gestos de mão em tempo real com Python e acionamento de saídas em um Arduino. A solução usa webcam, detecção de mãos com `cvzone`/MediaPipe e integração com a placa por dois caminhos: comunicação serial tradicional ou `pyFirmata`.

![Organização do hardware](pyfirmata_usage/hardware_organization.jpg)

## Visão geral

O sistema captura vídeo da webcam, identifica uma mão por quadro e traduz a combinação de dedos levantados em comandos para o Arduino. Cada gesto representa uma quantidade de dedos visíveis e pode acionar LEDs ou outras saídas digitais conectadas à placa.

O repositório contém duas abordagens de integração:

- `serial_version.py`: envia caracteres (`a` a `e`) via porta serial para um sketch Arduino dedicado.
- `pyfirmata_usage/`: controla pinos digitais do Arduino diretamente via protocolo Firmata.

## Casos de uso

- Protótipos de interface gestual com hardware.
- Demonstrações educacionais de visão computacional com Arduino.
- Acionamento simples de LEDs, relés ou sinais digitais a partir de gestos.
- Base inicial para automação hands-free.

## Arquitetura da solução

1. A webcam é lida com OpenCV.
2. O `HandDetector` da biblioteca `cvzone` identifica a mão e calcula os dedos levantados.
3. O software mapeia o gesto para um estado lógico.
4. O comando é enviado ao Arduino:
   - por serial, com mensagens curtas;
   - ou por `pyFirmata`, escrevendo diretamente nos pinos.
5. O Arduino responde acionando as saídas configuradas.

## Estrutura do projeto

```text
.
├── receiverRecognition.ino          # Sketch Arduino para modo serial
├── serial_version.py                # Aplicação Python usando serial
├── serial_version_details.txt       # Observação de uso do modo serial
├── requirements.txt                 # Dependências Python
├── pyfirmata_usage/
│   ├── main.py                      # Aplicação Python usando pyFirmata
│   ├── controller.py                # Mapeamento de gestos para pinos
│   ├── firmata_details.txt          # Observações do modo Firmata
│   └── hardware_organization.jpg    # Foto/diagrama da montagem
└── LICENSE
```

## Dependências

Dependências Python declaradas em `requirements.txt`:

- `opencv-python~=4.9.0.80`
- `cvzone~=1.6.1`
- `pyFirmata~=1.1.0`
- `pyserial~=3.5`
- `mediapipe~=0.10.14`

## Pré-requisitos

- Python 3.7 a 3.10 para melhor compatibilidade com `pyFirmata` 1.1.0.
- Arduino compatível com comunicação serial/Firmata.
- Webcam conectada ao computador.
- Arduino IDE para gravar o sketch ou `StandardFirmata`.
- LEDs e resistores, caso deseje reproduzir a demonstração de hardware.

## Instalação

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Mapeamento dos gestos

O reconhecimento considera uma mão e interpreta os dedos levantados nesta sequência:

- `[0, 0, 0, 0, 0]`: nenhum dedo levantado
- `[0, 1, 0, 0, 0]`: um dedo
- `[0, 1, 1, 0, 0]`: dois dedos
- `[0, 1, 1, 1, 0]`: três dedos
- `[0, 1, 1, 1, 1]`: quatro dedos
- `[1, 1, 1, 1, 1]`: cinco dedos

## Modo 1: integração via serial

### Como funciona

O arquivo `serial_version.py` abre a webcam, detecta a mão e envia caracteres pela serial apenas quando o gesto muda, evitando repetição contínua de comandos. O sketch `receiverRecognition.ino` lê os caracteres recebidos e aciona um pino digital por 1 segundo.

### Configuração

Antes de executar, ajuste:

- A porta serial em `serial_version.py`:
  - padrão atual: `"/dev/ttyACM0"`
- A câmera em `serial_version.py`:
  - padrão atual: `cv2.VideoCapture(0)`

### Carga no Arduino

Grave o arquivo `receiverRecognition.ino` na placa usando a Arduino IDE.

### Pinos usados no Arduino

No modo serial, os comandos acionam:

- `a` -> pino digital `2`
- `b` -> pino digital `3`
- `c` -> pino digital `4`
- `d` -> pino digital `5`
- `e` -> pino digital `12`

### Execução

```bash
python serial_version.py
```

Pressione `q` para encerrar.

## Modo 2: integração via pyFirmata

### Como funciona

O diretório `pyfirmata_usage/` usa o protocolo Firmata para controlar diretamente os pinos do Arduino a partir do Python, sem um sketch customizado de leitura serial. O arquivo `main.py` detecta os gestos e delega ao `controller.py` a escrita nos pinos digitais.

### Configuração

Antes de executar, ajuste:

- A porta da placa em `pyfirmata_usage/controller.py`
  - padrão atual: `COM7`
- A câmera em `pyfirmata_usage/main.py`
  - padrão atual: `cv2.VideoCapture(0)`

### Carga no Arduino

Na Arduino IDE, carregue o exemplo:

`Examples -> Firmata -> StandardFirmata`

### Pinos usados no Arduino

No modo `pyFirmata`, o projeto usa:

- `d:8:o`
- `d:9:o`
- `d:10:o`
- `d:11:o`
- `d:12:o`

Cada gesto liga progressivamente mais saídas.

### Execução

```bash
cd pyfirmata_usage
python main.py
```

Pressione `k` para encerrar.

## Observações técnicas encontradas no código

- O projeto está preparado para detectar no máximo uma mão por vez (`maxHands=1`).
- A confiança de detecção está configurada em `0.8`, privilegiando precisão sobre sensibilidade.
- O quadro é espelhado horizontalmente com `cv2.flip(frame, 1)`, melhorando a experiência de uso.
- No modo serial, existe uma proteção contra reenvio do mesmo gesto consecutivamente.
- Os textos exibidos na tela estão em português.

## Compatibilidade e atenção com ambiente

- O modo serial está configurado para Linux por padrão (`/dev/ttyACM0`).
- O modo `pyFirmata` está configurado para Windows por padrão (`COM7`).
- Isso indica que o repositório foi testado em contextos diferentes e exige ajuste manual da porta conforme o sistema operacional.

## Limitações atuais

- Não há arquivo de configuração central para portas e câmera.
- Não há tratamento explícito para falha de abertura da webcam ou da serial.
- O mapeamento de gestos é fixo e cobre apenas seis estados.
- O `pyFirmata` mencionado no projeto pode exigir ajuste manual por compatibilidade com versões mais novas do Python.

## Correção conhecida para pyFirmata

O arquivo `pyfirmata_usage/firmata_details.txt` registra uma incompatibilidade comum da biblioteca:

- ao executar pela primeira vez, pode ocorrer erro relacionado a `inspect.getargspec`
- a orientação documentada no próprio projeto é trocar esse uso por `inspect.getfullargspec`

Se possível, prefira executar esse modo com Python 3.7 a 3.10.

## Como adaptar para seu hardware

- Troque a porta serial conforme sua placa.
- Ajuste o índice da câmera se houver mais de um dispositivo conectado.
- Altere os pinos no sketch Arduino ou no `controller.py` para refletir sua montagem.
- Substitua LEDs por relés, buzzers ou outros atuadores digitais, respeitando os limites elétricos da placa.
