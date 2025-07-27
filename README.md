import pygame
import random

# Initialisation
pygame.init()
largeur, hauteur = 600, 400
taille = 20
ecran = pygame.display.set_mode((largeur, hauteur))
pygame.display.set_caption("🐍 Snake avec Sprite + Wrap-Around et Murs")

# Couleurs et outils
NOIR = (0, 0, 0)
ROUGE_MUR = (200, 0, 0)
clock = pygame.time.Clock()
police = pygame.font.SysFont(None, 36)

# Chargement des images avec gestion d'erreur
try:
    image_serpent = pygame.image.load("serpent.png").convert_alpha()
    image_serpent = pygame.transform.scale(image_serpent, (taille, taille))
    print("Image serpent chargée:", image_serpent.get_size())
except Exception as e:
    print("Erreur chargement serpent.png:", e)
    image_serpent = pygame.Surface((taille, taille))
    image_serpent.fill((0, 255, 0))  # Vert par défaut

try:
    image_pomme = pygame.image.load("pomme.png").convert_alpha()
    image_pomme = pygame.transform.scale(image_pomme, (taille, taille))
    print("Image pomme chargée:", image_pomme.get_size())
except Exception as e:
    print("Erreur chargement pomme.png:", e)
    image_pomme = pygame.Surface((taille, taille))
    image_pomme.fill((255, 0, 0))  # Rouge par défaut

# Positions des murs (x, y)
murs = [
    (100, 100),
    (140, 100),
    (180, 100),
    (300, 200),
    (300, 240),
    (300, 280),
]

# Dessine les murs
def dessiner_murs():
    for (x_mur, y_mur) in murs:
        pygame.draw.rect(ecran, ROUGE_MUR, (x_mur, y_mur, taille, taille))

# Dessine le serpent
def dessiner_serpent(corps):
    for bloc in corps:
        ecran.blit(image_serpent, (bloc[0], bloc[1]))

# Affiche le score
def afficher_score(score):
    texte = police.render(f"Score : {score}", True, (255, 255, 255))
    ecran.blit(texte, (10, 10))

# Affiche le message Game Over
def afficher_game_over():
    texte = police.render("Game Over - Appuie sur ESPACE pour rejouer", True, (255, 255, 255))
    ecran.blit(texte, (largeur // 2 - 200, hauteur // 2))
    pygame.display.update()

# Fonction principale du jeu
def boucle_jeu():
    x, y = largeur // 2, hauteur // 2
    dx, dy = taille, 0
    serpent = []
    longueur = 1
    score = 0

    pomme = [random.randrange(0, largeur, taille), random.randrange(0, hauteur, taille)]

    en_cours = True
    while en_cours:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_LEFT and dx == 0:
                    dx = -taille
                    dy = 0
                elif event.key == pygame.K_RIGHT and dx == 0:
                    dx = taille
                    dy = 0
                elif event.key == pygame.K_UP and dy == 0:
                    dx = 0
                    dy = -taille
                elif event.key == pygame.K_DOWN and dy == 0:
                    dx = 0
                    dy = taille

        x += dx
        y += dy

        # Traversée des murs (wrap-around)
        if x < 0:
            x = largeur - taille
        elif x >= largeur:
            x = 0
        if y < 0:
            y = hauteur - taille
        elif y >= hauteur:
            y = 0

        # Collision avec mur
        if (x, y) in murs:
            return  # Game over

        # Collision avec soi-même
        if [x, y] in serpent:
            return  # Game over

        serpent.append([x, y])
        if len(serpent) > longueur:
            serpent.pop(0)

        # Mange une pomme
        if [x, y] == pomme:
            longueur += 1
            score += 1
            # Regénère la pomme dans une position libre (hors murs et serpent)
            while True:
                pomme = [random.randrange(0, largeur, taille), random.randrange(0, hauteur, taille)]
                if pomme not in serpent and (pomme[0], pomme[1]) not in murs:
                    break

        # Affichage
        ecran.fill(NOIR)
        afficher_score(score)
        dessiner_murs()
        dessiner_serpent(serpent)
        ecran.blit(image_pomme, (pomme[0], pomme[1]))
        pygame.display.update()
        clock.tick(10)

# Boucle principale (rejouer)
while True:
    boucle_jeu()
    afficher_game_over()
    attente = True
    while attente:
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                exit()
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                attente = False
