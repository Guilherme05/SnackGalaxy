# Snack Galaxy

Snack Galaxy é um jogo simples desenvolvido com Pygame onde o jogador controla um personagem que coleta itens para aumentar sua pontuação e tamanho. O jogo possui uma tela de início onde o jogador pode selecionar a cor do personagem e um fundo que parece um espaço sideral.

## Requisitos

- Python 3.x
- Pygame

## Instalação

1. Certifique-se de ter o Python 3.x instalado em sua máquina.
2. Instale o Pygame usando o pip:
   ```bash
   pip install pygame
   ```
3. Coloque a imagem `espaco_sideral.jpg` no mesmo diretório do script `game.py`.

## Como jogar

1. Execute o script `game.py`:
   ```bash
   python game.py
   ```
2. Na tela de início, use as setas para cima e para baixo para selecionar a cor do seu personagem e pressione Enter para iniciar o jogo.
3. Use as setas do teclado para mover o personagem.
4. Colete os itens quadrados para aumentar sua pontuação e tamanho.
5. Quando o personagem atingir 50% da largura da tela, itens circulares aparecerão. Colete esses itens para retornar ao tamanho padrão.
6. Se o personagem sair da tela, o jogo termina. Pressione Enter para reiniciar.

## Estrutura do código

### Importações e inicialização

```python
import pygame
import sys
import random

pygame.init()
```

### Configuração da tela e carregamento da imagem de fundo

```python
screen = pygame.display.set_mode((800, 600))
pygame.display.set_caption("Snack Galaxy")

background = pygame.image.load('espaco_sideral.jpg')
background = pygame.transform.scale(background, (800, 600))
```

### Definição de cores

```python
BRANCO = (255, 255, 255)
AZUL = (0, 0, 255)
VERMELHO = (255, 0, 0)
VERDE = (0, 255, 0)
AMARELO = (255, 255, 0)
ROXO = (255, 0, 255)
CIANO = (0, 255, 255)
```

### Lista de cores possíveis para os itens

```python
cores_itens = [VERMELHO, VERDE, AMARELO, ROXO, CIANO]
```

### Fonte para exibir texto

```python
font = pygame.font.SysFont(None, 36)
```

### Função para exibir a tela de início

```python
def show_start_screen():
    selected_color = AZUL
    color_options = [AZUL, VERMELHO, VERDE, AMARELO, ROXO, CIANO]
    color_names = ["AZUL", "VERMELHO", "VERDE", "AMARELO", "ROXO", "CIANO"]
    selected_index = 0

    while True:
        screen.fill(BRANCO)
        title_text = font.render('Selecione a cor do seu personagem', True, (0, 0, 0))
        screen.blit(title_text, (200, 100))

        for i, color in enumerate(color_options):
            color_text = font.render(color_names[i], True, color)
            screen.blit(color_text, (350, 200 + i * 50))

        selected_color_text = font.render('>', True, (0, 0, 0))
        screen.blit(selected_color_text, (320, 200 + selected_index * 50))

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    selected_index = (selected_index - 1) % len(color_options)
                if event.key == pygame.K_DOWN:
                    selected_index = (selected_index + 1) % len(color_options)
                if event.key == pygame.K_RETURN:
                    return color_options[selected_index]
```

### Função para reiniciar o jogo

```python
def reset_game(player_color):
    global x, y, speed, item_positions, score, game_over, last_spawn_time, player_width, player_color_global, circle_items
    x, y = 400, 300
    speed = 5
    player_width = 50
    item_positions = [(random.randint(0, 750), random.randint(0, 550), random.choice(cores_itens)) for _ in range(5)]
    circle_items = []
    score = 0
    game_over = False
    last_spawn_time = pygame.time.get_ticks()
    player_color_global = player_color
```

### Loop principal do jogo

```python
# Inicializar o jogo
player_color = show_start_screen()
reset_game(player_color)

# Temporizador para gerar novos itens
item_spawn_time = 2000  # Tempo em milissegundos (2 segundos)

# Loop principal do jogo
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        if event.type == pygame.KEYDOWN and game_over:
            if event.key == pygame.K_RETURN:
                reset_game(player_color)

    if not game_over:
        # Capturar eventos de teclado
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            x -= speed
        if keys[pygame.K_RIGHT]:
            x += speed
        if keys[pygame.K_UP]:
            y -= speed
        if keys[pygame.K_DOWN]:
            y += speed

        # Verificar colisões com itens
        player_rect = pygame.Rect(x, y, player_width, 50)
        for item in item_positions[:]:
            item_rect = pygame.Rect(item[0], item[1], 20, 20)
            if player_rect.colliderect(item_rect):
                item_positions.remove(item)
                score += 1
                speed += 1  # Aumentar a velocidade a cada coleta de item
                player_width += 10  # Aumentar a largura do jogador a cada coleta de item

        # Verificar colisões com itens circulares
        for circle in circle_items[:]:
            circle_rect = pygame.Rect(circle[0] - 10, circle[1] - 10, 20, 20)
            if player_rect.colliderect(circle_rect):
                circle_items.remove(circle)
                player_width = 50  # Retornar o jogador ao tamanho padrão

        # Verificar se o jogador passou da borda da tela
        if x < 0 or x > 800 - player_width or y < 0 or y > 550:
            game_over = True

        # Gerar novos itens automaticamente
        current_time = pygame.time.get_ticks()
        if current_time - last_spawn_time > item_spawn_time:
            item_positions.append((random.randint(0, 750), random.randint(0, 550), random.choice(cores_itens)))
            last_spawn_time = current_time

        # Gerar itens circulares se o jogador tiver 50% da largura da tela
        if player_width >= 400 and len(circle_items) == 0:
            circle_items.append((random.randint(0, 750), random.randint(0, 550)))

    # Atualizar a tela
    screen.blit(background, (0, 0))  # Desenhar a imagem de fundo
    pygame.draw.rect(screen, player_color_global, (x, y, player_width, 50))
    for item in item_positions:
        pygame.draw.rect(screen, item[2], (item[0], item[1], 20, 20))
    for circle in circle_items:
        pygame.draw.circle(screen, (0, 0, 0), (circle[0], circle[1]), 10)

    # Exibir a pontuação
    score_text = font.render(f'Score: {score}', True, BRANCO)
    screen.blit(score_text, (10, 10))

    # Exibir mensagem de game over
    if game_over:
        game_over_text = font.render('Game Over', True, BRANCO)
        screen.blit(game_over_text, (350, 250))
        restart_text = font.render('Pressione Enter para reiniciar', True, BRANCO)
        screen.blit(restart_text, (250, 300))

    pygame.display.flip()

    # Controlar a taxa de atualização
    pygame.time.Clock().tick(30)
```

## Contribuição

Sinta-se à vontade para contribuir com melhorias para este jogo. Faça um fork do repositório, crie uma branch para suas alterações e envie um pull request.

## Licença

Este projeto está licenciado sob a licença MIT. Veja o arquivo LICENSE para mais detalhes.
