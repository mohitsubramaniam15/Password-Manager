from cryptography.fernet import Fernet
import pygame
import random
import sys

# Key generation and loading
def write_key():
    key = Fernet.generate_key()
    with open("key.key", "wb") as key_file:
        key_file.write(key)
write_key()

def load_key():
    with open("key.key", "rb") as file:
        return file.read()

key = load_key()
fer = Fernet(key)

# Password management functions
def view():
    try:
        with open('passwords.txt', 'r') as f:
            for line in f.readlines():
                data = line.rstrip()
                user, passw = data.split("|")
                print("User:", user, "| Password:", fer.decrypt(passw.encode()).decode())
    except FileNotFoundError:
        print("No passwords saved yet.")

def add():
    name = input("Account name: ")
    pwd = input("Password: ")
    with open('passwords.txt', 'a') as f:
        f.write(name + "|" + fer.encrypt(pwd.encode()).decode() + "\n")

# Pygame Snake Game
def snake_game():
    pygame.init()
    screen = pygame.display.set_mode((600, 400))
    pygame.display.set_caption("Snake Game")
    
    # Colors and game settings
    white = (255, 255, 255)
    red = (255, 0, 0)
    black = (0, 0, 0)
    green = (0, 255, 0)
    snake_pos = [100, 50]
    snake_body = [[100, 50], [90, 50], [80, 50]]
    direction = "RIGHT"
    change_to = direction
    speed = 15
    clock = pygame.time.Clock()
    score = 0

    # Food
    food_pos = [random.randrange(1, (600 // 10)) * 10, random.randrange(1, (400 // 10)) * 10]
    food_spawn = True

    # Main game loop
    while True:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP and direction != "DOWN":
                    change_to = "UP"
                elif event.key == pygame.K_DOWN and direction != "UP":
                    change_to = "DOWN"
                elif event.key == pygame.K_LEFT and direction != "RIGHT":
                    change_to = "LEFT"
                elif event.key == pygame.K_RIGHT and direction != "LEFT":
                    change_to = "RIGHT"

        direction = change_to
        if direction == "UP":
            snake_pos[1] -= 10
        elif direction == "DOWN":
            snake_pos[1] += 10
        elif direction == "LEFT":
            snake_pos[0] -= 10
        elif direction == "RIGHT":
            snake_pos[0] += 10

        # Snake growing and collision with food
        snake_body.insert(0, list(snake_pos))
        if snake_pos == food_pos:
            score += 1
            food_spawn = False
        else:
            snake_body.pop()
        if not food_spawn:
            food_pos = [random.randrange(1, (600 // 10)) * 10, random.randrange(1, (400 // 10)) * 10]
        food_spawn = True

        # Game Over conditions
        if (snake_pos[0] < 0 or snake_pos[0] > 590 or
                snake_pos[1] < 0 or snake_pos[1] > 390 or
                snake_pos in snake_body[1:]):
            break

        # Draw elements
        screen.fill(black)
        for pos in snake_body:
            pygame.draw.rect(screen, green, pygame.Rect(pos[0], pos[1], 10, 10))
        pygame.draw.rect(screen, red, pygame.Rect(food_pos[0], food_pos[1], 10, 10))

        pygame.display.flip()
        clock.tick(speed)

    pygame.quit()

# Main loop for password manager with error counter
error_count = 0

while True:
    mode = input(
        "Would you like to add a new password or view existing ones (type 'view' or 'add'), or press 'q' to quit? ").lower()
    if mode == "q":
        print("Thank you for using Mohit's Password Manager.")
        break
    elif mode == "view":
        view()
    elif mode == "add":
        add()
    else:
        print("Invalid Mode!")
        error_count += 1

    # Check error count
    if error_count >= 3:
        play_game = input("Too many invalid attempts! Type 'qwerty' to play a game, or anything else to continue: ")
        if play_game.lower() == "qwerty":
            snake_game()
            error_count = 0  # Reset error count after game
