# Lie or Legend - Visuele Pygame Versie

import pygame
import sys
import random

# Initialiseer pygame
pygame.init()

# Constantes
WIDTH, HEIGHT = 800, 600
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
FONT = pygame.font.SysFont('arial', 28)
BIGFONT = pygame.font.SysFont('arial', 36)

player_colors = {
    "Lisa": (255, 105, 180),
    "Daan": (100, 149, 237),
    "Bakker": (60, 179, 113)
}

# Achtergronden met gradient
def generate_backgrounds():
    backgrounds = []
    gradient_colors = [((255, 204, 229), (255, 255, 255)),
                       ((204, 229, 255), (255, 255, 255)),
                       ((204, 255, 229), (255, 255, 255))]
    for top_color, bottom_color in gradient_colors:
        surface = pygame.Surface((WIDTH, HEIGHT))
        for y in range(HEIGHT):
            ratio = y / HEIGHT
            color = [
                int(top_color[i] * (1 - ratio) + bottom_color[i] * ratio)
                for i in range(3)
            ]
            pygame.draw.line(surface, color, (0, y), (WIDTH, y))
        backgrounds.append(surface)
    return backgrounds

backgrounds = generate_backgrounds()

# Venster
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Lie or Legend")

# Speldata
story_prompts = [
    "Vertel iets gênants",
    "Iets wat misging op vakantie",
    "Een rare hobby",
    "Een beroemd persoon die je zogenaamd hebt ontmoet",
    "Een geheim talent dat niemand kent",
    "Een situatie waarin je moest improviseren",
    "Een keer dat je werd betrapt"
]

lisa_verhalen = [
    "Ik viel van het podium tijdens de schoolmusical.",
    "Op vakantie at ik per ongeluk hondenvoer omdat ik dacht dat het een snack was.",
    "Mijn hobby is het maken van miniatuurversies van fastfood in klei."
]
daan_verhalen = [
    "Ik heb ooit een game-toernooi gewonnen zonder te trainen.",
    "Ik gaf een presentatie in verzonnen Spaans.",
    "Mijn geheime talent is beatboxen met mijn neus."
]
bakker_verhalen = [
    "Ik dacht dat TikTok een klok-app was.",
    "Ik deed alsof ik een Fortnite-expert was.",
    "Ik debatteerde tegen mezelf in de klas."
]

players = ["Lisa", "Daan", "Bakker"]
scores = {naam: 0 for naam in players}
current_round = 1
max_rounds = 3

# Tekst tekenen

def draw_text(text, x, y, color=BLACK, font=FONT):
    surface = font.render(text, True, color)
    screen.blit(surface, (x, y))

def draw_bubble_text(text, x, y, width=700, padding=20, color=(255, 255, 255), text_color=BLACK):
    words = text.split(' ')
    lines = []
    current_line = ''
    for word in words:
        test_line = current_line + word + ' '
        if FONT.size(test_line)[0] < width - 2 * padding:
            current_line = test_line
        else:
            lines.append(current_line.strip())
            current_line = word + ' '
    lines.append(current_line.strip())

    bubble_height = padding * 2 + len(lines) * FONT.get_height()
    bubble_rect = pygame.Rect(x, y, width, bubble_height)
    pygame.draw.rect(screen, color, bubble_rect, border_radius=20)
    pygame.draw.rect(screen, (180, 180, 180), bubble_rect, 2, border_radius=20)

    for i, line in enumerate(lines):
        txt_surface = FONT.render(line, True, text_color)
        screen.blit(txt_surface, (x + padding, y + padding + i * FONT.get_height()))

def draw_header(title):
    pygame.draw.rect(screen, (50, 50, 120), (0, 0, WIDTH, 60))
    draw_text(title, 20, 15, color=(255, 255, 255), font=BIGFONT)

def wait_for_key():
    waiting = True
    while waiting:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            elif event.type == pygame.KEYDOWN:
                waiting = False

def show_intro_screen():
    bg = random.choice(backgrounds)
    screen.blit(bg, (0, 0))
    draw_header("Welkom bij Lie or Legend!")
    draw_bubble_text("In dit partyspel vertelt telkens één speler een verhaal. Maar spreekt diegene de waarheid of liegt die? Stem Lie of Legend!", 50, 100)
    draw_text("Druk op een toets om te beginnen!", 200, 400, color=(80, 80, 80))
    pygame.display.flip()
    wait_for_key()

def choose_prompt():
    return random.choice(story_prompts)

def choose_ai_story(name):
    if name == "Lisa":
        return random.choice(lisa_verhalen)
    elif name == "Daan":
        return random.choice(daan_verhalen)
    elif name == "Bakker":
        return random.choice(bakker_verhalen)
    return "Automatisch gegenereerd verhaal."

def play_round(storyteller):
    bg = random.choice(backgrounds)
    screen.blit(bg, (0, 0))
    draw_header("Lie or Legend")
    draw_text(f"{storyteller} is de Storyteller!", 50, 70)

    prompt = choose_prompt()
    draw_bubble_text(f"Uitspraak: {prompt}", 50, 120)
    pygame.display.flip()
    wait_for_key()

    true_story = random.choice([True, False])
    verhaal = choose_ai_story(storyteller)

    screen.blit(bg, (0, 0))
    draw_header("Verhaal")
    draw_bubble_text(verhaal, 50, 120)
    pygame.display.flip()
    wait_for_key()

    votes = {}
    for p in players:
        if p != storyteller:
            votes[p] = random.choice([True, False])

    screen.blit(bg, (0, 0))
    draw_header("Resultaten Lie or Legend")
    y = 100
    for p, vote in votes.items():
        if vote == true_story:
            draw_text(f"{p} had het goed! +1 punt", 50, y, color=player_colors.get(p, BLACK))
            scores[p] += 1
        else:
            draw_text(f"{p} zat fout. Je bent erin getrapt!", 50, y, color=player_colors.get(p, BLACK))
        y += 40

    if not true_story:
        fooled = sum(1 for v in votes.values() if v)
        if fooled > 0:
            draw_text(f"Nice bluff, {storyteller}! +2 punten", 50, y)
            scores[storyteller] += 2
    else:
        not_believed = sum(1 for v in votes.values() if not v)
        if not_believed == len(players) - 1:
            draw_text(f"Niemand geloofde {storyteller}, maar het was waar! +3 punten", 50, y)
            scores[storyteller] += 3

    pygame.display.flip()
    wait_for_key()

def show_scores():
    screen.fill(WHITE)
    draw_header("Eindscore")
    y = 100
    for naam, score in scores.items():
        draw_text(f"{naam}: {score} punten", 50, y, color=player_colors.get(naam, BLACK))
        y += 40
    winnaar = max(scores, key=scores.get)
    draw_text(f"Winnaar: {winnaar}!", 50, y + 40, color=player_colors.get(winnaar, BLACK))
    pygame.display.flip()
    wait_for_key()

def start_game():
    show_intro_screen()
    for round_nr in range(max_rounds):
        for storyteller in players:
            play_round(storyteller)
    show_scores()

if __name__ == "__main__":
    start_game()
    pygame.quit()
