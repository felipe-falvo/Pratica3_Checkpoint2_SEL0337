# SEL0337 — Projetos em Sistemas Embarcados
## Prática 3: Introdução à programação de alto nível com GPIO
### Checkpoint 2 — PWM e Aplicação com Sensor Ultrassônico

**Integrantes:**
- Felipe Assis Bernardes Falvo — Nº USP: 15004433
- Kayke Malaquias Gregorio — Nº USP: 15651561

---

## 1. Introdução

Esse GitHub tem a resolução do **Checkpoint 2** da Prática 3, dividido em duas partes:

1. Implementação de **PWM**, utilizada para controlar a intensidade de um LED;
2. Desenvolvimento de um **sistema de proximidade**, utilizando um sensor ultrassônico HC-SR04, um LED e um buzzer.

Na primeira parte, foi utilizada a biblioteca `RPi.GPIO` para gerar um sinal PWM no GPIO 18. O *duty cycle* é alterado gradualmente, fazendo com que o brilho do LED aumente e diminua. O sinal também foi observado utilizando um osciloscópio.

Na segunda parte, foi utilizada a biblioteca `gpiozero` para realizar a leitura do sensor HC-SR04. Quando um objeto é detectado a uma distância inferior a 30 cm, o LED e o buzzer são acionados.

---

# Parte 1 — PWM

## 2. Objetivo

Implementar um sinal **PWM (Pulse Width Modulation)** utilizando a Raspberry Pi para controlar a intensidade luminosa de um LED.

O *duty cycle* representa a porcentagem do período em que o sinal permanece em nível alto.

- **0%:** LED desligado;
- **50%:** sinal em nível alto durante metade do período;
- **100%:** LED ligado continuamente.

---

## 3. Montagem

O LED foi conectado ao **GPIO 18** da Raspberry Pi através de um resistor e montado em uma protoboard.

### Componentes

- Raspberry Pi;
- Protoboard;
- LED;
- Resistor;
- Jumpers;
- Osciloscópio.

O GPIO foi configurado utilizando a numeração **BCM**.

---

## 4. Configuração do PWM

Foi utilizada a biblioteca `RPi.GPIO`.

O GPIO 18 foi configurado como saída e o PWM foi iniciado com frequência de **50 Hz**:

```python
import RPi.GPIO as GPIO
import time

pino_LED = 18

GPIO.setmode(GPIO.BCM)
GPIO.setup(pino_LED, GPIO.OUT)

led_com_pwm = GPIO.PWM(pino_LED, 50)
led_com_pwm.start(0)
```

## 5. Variação do Duty Cycle

O programa aumenta o *duty cycle* de 0% até 100%, em passos de 5%:

```python
for i in range(0, 101, 5):
    led_com_pwm.ChangeDutyCycle(i)
    time.sleep(0.5)
```

Depois, o valor é reduzido de 100% até 0%:

```python
for i in range(100, -1, -5):
    led_com_pwm.ChangeDutyCycle(i)
    time.sleep(0.5)
```

Esse processo é repetido.

Assim, o LED fica aumentando e diminuindo o brilho.

---

## 6. Código — `pwm.py`

O código completo utilizado está no arquivo `PWM_led/pwm.py`.

```python
import RPi.GPIO as GPIO # biblioteca para os pinos da raspberry
import time # biblioteca de tempo

# definindo o pino 18 o LED
pino_LED = 18

# configurando para broadcom (BCM)
GPIO.setmode(GPIO.BCM)

# configurando o pino do LED como saída (OUTPUT)
GPIO.setup(pino_LED, GPIO.OUT)

# configurando o pwm no pino do LED com frequência de 50 Hz
led_com_pwm = GPIO.PWM(pino_LED, 50)

# inicia o pwm com um duty cycle de 0% (LED começa desligado)
led_com_pwm.start(0)

print("Aperta CRTL+C para desligar")


try:
    # manter o ciclo do pwn funcionando
    while True:
    
        # aumenta o brilho de 0 a 100 com incrementos de 5
        for i in range(0, 101, 5):
            led_com_pwm.ChangeDutyCycle(i) # Atualiza o duty cycle do PWM
            time.sleep(0.5) # 0.5 segundos para cada intensidade
            
        # diminui o brilho de 100 a 0 com decrementos de 5    
        for i in range(100, -1, -5):
            led_com_pwm.ChangeDutyCycle(i) # Atualiza o duty cycle do PWM
            time.sleep(0.5) # 0.5 segundos para cada intensidade

# programa desliga quando aperta CTRL+C
except:
    led_com_pwm.stop() # para com o sinal pwm
    GPIO.cleanup() # limpa as configurações feitas durante o código
```

---

## 7. Visualização no Osciloscópio

O sinal PWM foi observado utilizando um osciloscópio.

Com a frequência de 50 Hz, o período do sinal é de aproximadamente 20 ms. Ao alterar o *duty cycle*, a frequência permanece a mesma, mas a largura do pulso é alterada.

Dessa forma, foi possível observar no osciloscópio a relação entre o *duty cycle* configurado no programa e o sinal produzido pelo GPIO.

---

## 8. Demonstração

A demonstração do funcionamento do PWM está disponível em:

```text
PWM_led/video_prova_pwm_led.mp4
```

O vídeo mostra a variação da intensidade luminosa do LED e a visualização do sinal PWM no osciloscópio.

---

# Parte 2 — Sistema de Proximidade

## 9. Objetivo

Foi desenvolvido um sistema de detecção de proximidade utilizando:

- sensor ultrassônico HC-SR04;
- LED;
- buzzer;
- Raspberry Pi.

O sensor realiza medições. Quando um objeto está a menos de **30 cm**, o LED e o buzzer são acionados.

---

## 10. Montagem

Os componentes foram conectados à Raspberry Pi da seguinte forma:

- **HC-SR04:** Echo no GPIO 24;
- **HC-SR04:** Trigger no GPIO 23;
- **LED:** GPIO 14;
- **Buzzer:** GPIO 25.

O circuito foi montado em uma protoboard.

---

## 11. Funcionamento

Foi utilizada a biblioteca `gpiozero` para configurar o sensor e os atuadores.

O sensor é configurado através de:

```python
sensor = DistanceSensor(echo=24, trigger=23, max_distance=2)
```

A distância medida pelo sensor é convertida de metros para centímetros:

```python
distancia_cm = sensor.distance * 100
```

O sistema verifica a distância:

```python
if distancia_cm < 30:
    ativar_alerta()
else:
    desativar_alerta()
```

Portanto:

- distância menor que 30 cm o LED e buzzer ficam ligados
- distância igual ou maior que 30 cm o LED e buzzer ficam desligados

---

## 12. Código — `projeto.py`

O código completo está no arquivo `projeto/projeto.py`.

```python
from gpiozero import DistanceSensor, LED, Buzzer # importando as bibliotecas do gpiozero
import time # importando a biblioteca de tempo

# configurando o sensor HC-SR04 nos pinos 24 (Echo) e 23 (Trigger) 
# max_distance -> indica que a distância máxima de alcance é de 2 metros
sensor = DistanceSensor(echo = 24, trigger=23, max_distance = 2)

# configurando o LED no pino 14
led = LED(14)

# configurando o buzzer no pino 25
buzzer = Buzzer(25)

print("Inicio:\n")

# função modularizada para ligar o LED e o Buzzer
def ativar_alerta():
    led.on() # acender o LED
    buzzer.on() # ligar o buzzer

# função modularizada para desligar o LED e o Buzzer
def desativar_alerta():
    led.off() # apagar o LED
    buzzer.off() # desligar o buzzer


try:
    # laço da leitura do sensor
    while True:
        time.sleep(0.1) # Aguarda 0.1 segundo entre cada medição
        
        distancia_cm = sensor.distance * 100 # Converte a distância de metros para centímetros
        
        # Mostra a distância atualizando na mesma linha (\r)
        print(f"\rdistância :{distancia_cm:.1f} cm", end="", flush=True)

        # Condições:
        # Se o objeto estiver a menos de 30 cm, liga o buzzer e o LED
        if distancia_cm < 30:
            ativar_alerta()
        # Caso contrário, o buzzer e o LED ficam desligados
        else:
            desativar_alerta()

# comando CTRL+C no terminal para desligar o programa
except KeyboardInterrupt:
    desativar_alerta() # garantindo que o LED e o buzzer desliguem ao parar o código
    print("\n")
```

---

## 13. Demonstração

A demonstração do código e o hardware está em:

```text
projeto/video_prova.mp4
```

O vídeo mostra a leitura da distância e o acionamento do LED e do buzzer quando um objeto se aproxima do sensor.

A foto do hardware está em:

```text
projeto/circuito.jpeg
```


---

## 14. Formato de Entrega — Checkpoint 2

De acordo com o roteiro da disciplina, a entrega do Checkpoint 2 consiste em:

- enviar os **dois scripts Python `.py`**, com as linhas de código comentadas:
  - `pwm.py`, referente à implementação do PWM;
  - `projeto.py`, referente à aplicação escolhida;
- alternativamente, pode ser enviado um **link para o repositório do GitHub** contendo os arquivos;
- o link pode ser enviado em um arquivo de texto;
- realizar o upload dos programas diretamente na tarefa do **e-Disciplinas** até a data de entrega.

Nesse repositório, os dois scripts estão disponíveis nos diretórios:

```text
PWM_led/pwm.py
projeto/projeto.py
```

**Observação:** o roteiro de entrega do Checkpoint 2 solicita os dois scripts `.py` comentados e não pede a entrega de arquivos de histórico de execução. Portanto, **não foi necessário anexar histórico** para esta etapa.
