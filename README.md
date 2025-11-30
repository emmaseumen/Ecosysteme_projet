# BIENVENUE DANS NOTRE NOUVEAU PROJET #

# 🦒🦒 ECOSYSTEM_PROJET 🦕🦕 #

Ce projet est une petite simulation d un ecosysteme, ou des entites(animaux) se deplacent dans un monde 2D a la recherche de nourriture. Chaque entite possede une position, un comportement, et un systeme pour trouver la nourriture la plus proche.

## 👽👽 FONCTIONALITE PRINCIPALE ##
### Mouvement ###
Les entites utilisent un Vector2D(x, y) pour se deplacer

### Detection de nourriture ###
Chaque entite peut detecter la nourriture grace a la fonction SeekFood

# 📰📰 STRUCTURE DU PROGET #
ecosystem_simulator/ 
├── include/ 
│   ├── Core/ 
│   │   ├── Structs.hpp 
│   │   ├── Entity.hpp 
│   │   └── Ecosystem.hpp 
│   └── Graphics/ 
│       ├── Window.hpp 
│       └── Renderer.hpp 
├── src/ 
│   ├── Core/ 
│   │   ├── Entity.cpp 
│   │   └── Ecosystem.cpp 
│   ├── Graphics/ 
│   │   ├── Window.cpp 
│   │   └── Renderer.cpp 
│   └── main.cpp 
├── assets/ 
│   └── (futures textures) 
└── README.md 

#### Exemple de vector 2D ####
```
struct Vector2D { 
    float x; 
    float y; 
    // �� Constructeur avec valeurs par défaut 
    Vector2D(float x = 0.0f, float y = 0.0f) : x(x), y(y) {} 
    // �� Méthodes utilitaires 
    float Distance(const Vector2D& other) const { 
        float dx = x - other.x; 
        float dy = y - other.y; 
        return std::sqrt(dx * dx + dy * dy); 
    }
    Vector2D operator+(const Vector2D& other) const { 
        return Vector2D(x + other.x, y + other.y); 
    }
    Vector2D operator*(float scalar) const { 
        return Vector2D(x * scalar, y * scalar); 
}
```
