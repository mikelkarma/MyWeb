# Mapeamento do Ambiente com Sensor Ultrassônico e Servo Motor

Neste projeto, demonstramos como realizar o mapeamento do ambiente utilizando um sensor ultrassônico integrado a um servo motor e controlado por um Arduino. A ideia é acoplar o servo motor ao sensor ultrassônico. Quando o servo motor gira, ele movimenta o sensor, permitindo coletar dados sobre a distância em diferentes ângulos.

## Código Arduino

O código utilizado para esse projeto é o seguinte:

```cpp
#include <Servo.h>

const int trigPin = 11; // Pino para o Trigger do sensor ultrassônico
const int echoPin = 10; // Pino para o Echo do sensor ultrassônico

long tempo; // Variável para armazenar o tempo do pulso
int distancia; // Variável para armazenar a distância medida

Servo servo; // Criação do objeto servo

void setup() {
  pinMode(trigPin, OUTPUT); // Define o pino do Trigger como saída
  pinMode(echoPin, INPUT); // Define o pino do Echo como entrada
  Serial.begin(9600); // Inicia a comunicação serial
  servo.attach(12); // Conecta o servo ao pino 12
}

void loop() {
  // Gira o servo de 15° a 165°
  for (int i = 15; i <= 165; i++) {  
    servo.write(i); // Move o servo para a posição 'i'
    delay(30); // Aguarda 30 milissegundos
    distancia = calculoDistancia(); // Calcula a distância
    Serial.print(i); 
    Serial.print(","); 
    Serial.print(distancia); 
    Serial.println(); // Quebra de linha para facilitar a leitura
  }
  
  // Gira o servo de 165° a 15°
  for (int i = 165; i > 15; i--) {  
    servo.write(i); // Move o servo para a posição 'i'
    delay(30); // Aguarda 30 milissegundos
    distancia = calculoDistancia(); // Calcula a distância
    Serial.print(i);
    Serial.print(",");
    Serial.print(distancia);
    Serial.println(); // Quebra de linha para facilitar a leitura
  }
}

int calculoDistancia() { 
  digitalWrite(trigPin, LOW); // Garante que o Trigger esteja em LOW
  delayMicroseconds(2); // Espera 2 microssegundos
  digitalWrite(trigPin, HIGH); // Envia o pulso para o Trigger
  delayMicroseconds(10); // Mantém o pulso por 10 microssegundos
  digitalWrite(trigPin, LOW); // Finaliza o pulso

  tempo = pulseIn(echoPin, HIGH); // Lê o tempo de retorno do Echo
  distancia = tempo * 0.034 / 2; // Calcula a distância em centímetros
  return distancia; // Retorna a distância calculada
}
```

# Comunicação Serial
Os dados coletados são enviados via comunicação serial Bluetooth, onde são repassados a um código Python separado que processa as informações e as transforma em um mapa.

## Formato de Dados
> Envio: angulo,distancia.angulo2,distancia2

> Recebimento: codigopython -> mapa

```python
import numpy as np
import matplotlib.pyplot as plt
import pygame
import matplotlib.patches as patches

# Inicialização do Pygame
pygame.init()

# Configurações
size = 30  # Tamanho inicial do quadrado
step = 1   # Passo de movimento do carrinho
cart_pos = np.array([size / 2, size / 2])  # Posição inicial do carrinho
expand_threshold = 1  # Distância para aumentar o tamanho do mapa
objects = []  # Lista para armazenar objetos

# Criando a figura 2D
plt.ion()  # Modo interativo
fig, ax = plt.subplots()

# Função para adicionar um objeto com base no ângulo e distância
def add_object(angle, distance):
    x = cart_pos[0] + distance * np.cos(angle)
    y = cart_pos[1] + distance * np.sin(angle)
    if 0 <= x <= size and 0 <= y <= size:
        objects.append((x, y))

# Loop de visualização
running = True
while running:
    ax.clear()
    ax.scatter(cart_pos[0], cart_pos[1], color='blue', s=100)  # Carrinho

    # Desenha os objetos como quadrados
    for obj in objects:
        square = patches.Rectangle((obj[0] - 0.5, obj[1] - 0.5), 1, 1, color='red')  # Quadrado de 1x1
        ax.add_patch(square)

    # Desenha a grade
    ax.set_xticks(np.arange(0, size + 1, 1))
    ax.set_yticks(np.arange(0, size + 1, 1))
    ax.grid()
    
    for i in range(361):
        add_object(i, 10)
    
    # Limites do gráfico
    ax.set_xlim([0, size])
    ax.set_ylim([0, size])

    # Verifica se o carrinho está perto da borda
    if cart_pos[0] >= size - expand_threshold or cart_pos[0] <= expand_threshold or \
       cart_pos[1] >= size - expand_threshold or cart_pos[1] <= expand_threshold:
        size += 5  # Aumenta o tamanho do mapa

    # Atualiza o gráfico
    plt.pause(0.1)

    # Verifica eventos do Pygame
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

        # Movimentação do carrinho
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP and cart_pos[1] < size:
                cart_pos[1] += step
            elif event.key == pygame.K_DOWN and cart_pos[1] > 0:
                cart_pos[1] -= step
            elif event.key == pygame.K_LEFT and cart_pos[0] > 0:
                cart_pos[0] -= step
            elif event.key == pygame.K_RIGHT and cart_pos[0] < size:
                cart_pos[0] += step
            
            # Adiciona objeto com base no ângulo e distância
            elif event.key == pygame.K_a:
                angle = np.pi / 4  # Exemplo: 45 graus
                distance = 2       # Distância do carrinho
                add_object(angle, distance)  # Adiciona um objeto na direção especificada

pygame.quit()

```

Os objetos serão adicionados através da seguinte função Python:

```
def add_object(angle, distance):
    x = cart_pos[0] + distance * np.cos(angle)
    y = cart_pos[1] + distance * np.sin(angle)
    if 0 <= x <= size and 0 <= y <= size:
        objects.append((x, y))

```

Onde repassamos os dados do ângulo do Arduino e a distância captada pelo sensor ultrassônico, processando e transformando em um mapa.
