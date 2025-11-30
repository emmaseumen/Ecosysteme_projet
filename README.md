# BIENVENUE DANS NOTRE NOUVEAU PROJET #

# 🦒🦒 ECOSYSTEM_PROJET 🦕🦕 #

Ce projet est une petite simulation d un ecosysteme, ou des entites(animaux) se deplacent dans un monde 2D a la recherche de nourriture. Chaque entite possede une position, un comportement, et un systeme pour trouver la nourriture la plus proche.

## 👽👽 FONCTIONALITE PRINCIPALE ##
### Mouvement ###
Les entites utilisent un Vector2D(x, y) pour se deplacer

### Detection de nourriture ###
Chaque entite peut detecter la nourriture grace a la fonction SeekFood
```cpp
Vector2D Entity::SeekFood(const std::vector<Food>& foodSources) const {
            Vector2D closestPos(0.0f, 0.0f);
            float closestDist = std::numeric_limits<float>::max();

            for(const Food& food : foodSources) {
                float dist = position.Distance(food.position);
                if (dist < closestDist) {
                    closestDist = dist;
                    closestPos = food.position;
                }
            }
            
            float length = std::sqrt(closestPos.x * closestPos.x + closestPos.y * closestPos.y);
            if(length > 0.0f){
                closestPos.x /= length;
                closestPos.y /= length;
            }
            return closestPos;
        }
```

### pour rester dans la fenetre(StayInBoungt)
Cette fonction permet aux entites de rester dans la fenetre 
```cpp
Vector2D Entity::StayInBounds(float worldWidth, float worldHeight){
            
            if (position.x < 0) position.x = 0;
            if (position.y < 0) position.y = 0;
            
            if (position.x > worldWidth) position.x = worldWidth;
            if (position.y > worldHeight) position.y = 0;

            return position;
        }
```

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
```cpp
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

# 📰📰 FICHIER DU PROJET #
## Include/Core/Ecosystheme.h ##
```cpp
#pragma once

#include "Entity.h" 
#include "Structs.h" 
#include <vector> 
#include <memory> 
#include <random> 

namespace Ecosystem { 
    namespace Core { 
        class Ecosystem { 
        private: 
            //ÉTAT INTERNE 
            std::vector<std::unique_ptr<Entity>> mEntities; 
            std::vector<Food> mFoodSources; 
            float mWorldWidth; 
            float mWorldHeight; 
            int mMaxEntities; 
            int mDayCycle; 
            //Générateur aléatoire 
            mutable std::mt19937 mRandomGenerator; 
            //STATISTIQUES 
            struct Statistics { 
                int totalHerbivores; 
                int totalCarnivores; 
                int totalPlants; 
                int totalFood; 
                int deathsToday; 
                int birthsToday; 
            } mStats; 
        public: 
            //CONSTRUCTEUR/DESTRUCTEUR 
            Ecosystem(float width, float height, int maxEntities = 500); 
            ~Ecosystem(); 
            //MÉTHODES PUBLIQUES 
            void Initialize(int initialHerbivores, int initialCarnivores, int initialPlants);
            void Update(float deltaTime); 
            void SpawnFood(int count); 
            void RemoveDeadEntities(); 
            void HandleReproduction(); 
            void HandleEating(); 
            //GETTERS 
            int GetEntityCount() const { return mEntities.size(); } 
            int GetFoodCount() const { return mFoodSources.size(); } 
            Statistics GetStatistics() const { return mStats; } 
            float GetWorldWidth() const { return mWorldWidth; } 
            float GetWorldHeight() const { return mWorldHeight; } 
            // MÉTHODES DE GESTION 
            void AddEntity(std::unique_ptr<Entity> entity); 
            void AddFood(Vector2D position, float energy = 25.0f); 
            //RENDU 
            void Render(SDL_Renderer* renderer) const; 
        private: 
            //MÉTHODES PRIVÉES 
            void UpdateStatistics(); 
            void SpawnRandomEntity(EntityType type); 
            Vector2D GetRandomPosition() const; 
            void HandlePlantGrowth(float deltaTime); 
        }; 
    } // namespace Core 
} // namespace Ecosystem
```

## Include/Core/Entity.h ##
```cpp
#pragma once 
#include "Structs.h" 
#include <SDL3/SDL.h> 
#include <memory> 
#include "random" 
#include <vector> 
namespace Ecosystem { 
    namespace Core { 
        // ��ÉNUMÉRATION DES TYPES D'ENTITÉS 
        enum class EntityType { 
            HERBIVORE, 
            CARNIVORE, 
            PLANT 
            
        }; 
        class Entity { 
        private: 
            // ��DONNÉES PRIVÉES - État interne protégé 
            float mEnergy; 
            float mMaxEnergy; 
            int mAge; 
            int mMaxAge; 
            bool mIsAlive; 
            Vector2D mVelocity; 
            EntityType mType; 
            // ��Générateur aléatoire 
            mutable std::mt19937 mRandomGenerator; 
        public: 
            // ��DONNÉES PUBLIQUES - Accès direct sécurisé 
            Vector2D position; 
            Color color; 
            float size; 
            std::string name; 
            // �� CONSTRUCTEURS 
            Entity(EntityType type, Vector2D pos, std::string entityName = "Unnamed"); 
            Entity(const Entity& other);  // Constructeur de copie 
            // �� DESTRUCTEUR 
            ~Entity(); 
            // ⚙MÉTHODES PUBLIQUES 
            void Update(float deltaTime); 
            void Move(float deltaTime); 
            void Eat(float energy); 
            bool CanReproduce() const; 
            std::unique_ptr<Entity> Reproduce(); 
            void ApplyForce(Vector2D force); 
            // ��GETTERS - Accès contrôlé aux données privées 
            float GetEnergy() const { return mEnergy; } 
            float GetEnergyPercentage() const { return mEnergy / mMaxEnergy; } 
            int GetAge() const { return mAge; } 
            bool IsAlive() const { return mIsAlive; } 
            EntityType GetType() const { return mType; } 
            Vector2D GetVelocity() const { return mVelocity; } 
            // ��MÉTHODES DE COMPORTEMENT 
            Vector2D SeekFood(const std::vector<Food>& foodSources) const; 
            Vector2D AvoidPredators(const std::vector<Entity>& predators) const; 
            Vector2D StayInBounds(float worldWidth, float worldHeight); 
            // ��MÉTHODE DE RENDU 
            void Render(SDL_Renderer* renderer) const; 
        private: 
            // ��MÉTHODES PRIVÉES - Logique interne 
            void ConsumeEnergy(float deltaTime); 
            void Age(float deltaTime); 
            void CheckVitality(); 
            Vector2D GenerateRandomDirection(); 
            Color CalculateColorBasedOnState() const; 
        }; 
    } // namespace Core 
} // namespace Ecosystem 
```

## Include/Core/GameEngine.h
```cpp
#pragma once 
#include "../Graphics/Window.h" 
#include "Ecosystem.h" 
#include <chrono> 
namespace Ecosystem { 
namespace Core { 
class GameEngine { 
private: 
    // �� ÉTAT DU MOTEUR 
    Graphics::Window mWindow; 
    Ecosystem mEcosystem; 
    bool mIsRunning; 
    bool mIsPaused; 
    float mTimeScale; 
     
    // ⏱ CHRONOMÉTRE 
    std::chrono::high_resolution_clock::time_point mLastUpdateTime; 
    float mAccumulatedTime; 
public: 
    // 🏗 CONSTRUCTEUR 
    GameEngine(const std::string& title, float width, float height); 
     
    // ⚙ MÉTHODES PRINCIPALES 
    bool Initialize(); 
    void Run(); 
    void Shutdown(); 
     
    // �� GESTION D'ÉVÉNEMENTS 
    void HandleEvents(); 
    void HandleInput(SDL_Keycode key); 
private: 
    // �� MÉTHODES INTERNES 
    void Update(float deltaTime); 
    void Render(); 
    void RenderUI(); 
}; 
} // namespace Core 
} // namespace Ecosystem 
```

## Include/Core/Structs.h ##
```cpp
#pragma once 
#include <cstdint> 
#include <string> 
#include <cmath> 
namespace Ecosystem { 
    namespace Core { 
        // 🏷 STRUCTS POUR LES DONNÉES SIMPLES 
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
        }; // implementation de la structure vector2D
        struct Color { 
            uint8_t r; 
            uint8_t g; 
            uint8_t b; 
            uint8_t a; 
            // �� Constructeurs multiples 
            Color() : r(255), g(255), b(255), a(255) {}  // Blanc par défaut 
            Color(uint8_t red, uint8_t green, uint8_t blue, uint8_t alpha = 255)  
                : r(red), g(green), b(blue), a(alpha) {} 
            // �� Couleurs prédéfinies 
            static Color Red() { return Color(255, 0, 0); } 
            static Color Green() { return Color(0, 255, 0); } 
            static Color Blue() { return Color(0, 0, 255); } 
        }; 

            static Color Yellow() { return Color(255, 255, 0); } 
        struct Food { 
            Vector2D position; 
            float energyValue; 
            Color color; 
            // �� Constructeur 
            Food(Vector2D pos, float energy = 25.0f)  
                : position(pos), energyValue(energy), color(Color::Green()) {} 
        }; 
    } // namespace Core 
} // namespace Ecosystem 
```

## Include/Graphics/Renderer.h ##
```cpp
#pragma once 
#include <SDL3/SDL.h>
 
SDL_Init(SDL_INIT_VIDEO) ;

SDL_Window* window = SDL_CreateWindowAndRenderer("ecosystem simulator", 800, 600, 0);
```

## Include/Graphics/Window.h ##
```cpp
#pragma once 
#include <SDL3/SDL.h> 
#include <string> 
#include "../Core/Structs.h" 

namespace Ecosystem { 
    namespace Graphics { 
        class Window { 
        private: 
            // �� RESSOURCES SDL 
            SDL_Window* mWindow; 
            SDL_Renderer* mRenderer; 
            float mWidth; 
            float mHeight; 
            bool mIsInitialized; 
            std::string mTitle; 
        public: 
            // 🏗 CONSTRUCTEUR/DESTRUCTEUR 
            Window(const std::string& title, float width, float height); 
            ~Window(); 
            // ⚙ INITIALISATION 
            bool Initialize(); 
            void Shutdown(); 
            
            // �� RENDU 
            void Clear(const Core::Color& color = Core::Color(30, 30, 30)); 
            void Present(); 
            
            // �� GETTERS 
            SDL_Renderer* GetRenderer() const { return mRenderer; } 
            bool IsInitialized() const { return mIsInitialized; } 
            float GetWidth() const { return mWidth; } 
            float GetHeight() const { return mHeight; } 
            std::string GetTitle() const { return mTitle; } 
        }; 
    } // namespace Graphics 
} // namespace Ecosystem 

```

## src/Core/Ecosysteme.cpp ##
```cpp
#include "Core/Ecosystem.h" 
#include <algorithm> 
#include <iostream> 
namespace Ecosystem { 
    namespace Core { 
        // 🏗 CONSTRUCTEUR 
        Ecosystem::Ecosystem(float width, float height, int maxEntities) 
            : mWorldWidth(width), mWorldHeight(height), mMaxEntities(maxEntities), 
            mDayCycle(0), mRandomGenerator(std::random_device{}()) 
        { 
            // Initialisation des statistiques 
            mStats = {0, 0, 0, 0, 0, 0}; 
            std::cout << "🌍Écosystème créé: " << width << "x" << height << std::endl; 
        } 
        // 🗑 DESTRUCTEUR 
        Ecosystem::~Ecosystem() { 
            std::cout << "🌍Écosystème détruit (" << mEntities.size() << " entités nettoyé" << std::endl;
        } 
        // INITIALISATION 
        void Ecosystem::Initialize(int initialHerbivores, int initialCarnivores, int initialPlants) {
            mEntities.clear(); 
            mFoodSources.clear(); 
            // Création des entités initiales 
            for (int i = 0; i < initialHerbivores; ++i) { 
                SpawnRandomEntity(EntityType::HERBIVORE); 
            }
            for (int i = 0; i < initialCarnivores; ++i) { 
                SpawnRandomEntity(EntityType::CARNIVORE); 
            }
            for (int i = 0; i < initialPlants; ++i) { 
                SpawnRandomEntity(EntityType::PLANT); 
            }
            // Nourriture initiale 
            SpawnFood(20); 
            std::cout << "🌱Écosystème initialisé avec " << mEntities.size() << " entités" <<std::endl;
        } 
        // MISE À JOUR 
        void Ecosystem::Update(float deltaTime) { 
            // Mise à jour de toutes les entités 
            for (auto& entity : mEntities) { 
                entity->Update(deltaTime); 
            }
            // Gestion des comportements 
            HandleEating(); 
            HandleReproduction(); 
            RemoveDeadEntities(); 
            HandlePlantGrowth(deltaTime); 
            // Mise à jour des statistiques 
            UpdateStatistics(); 
            mDayCycle++; 
        } 
        // GÉNÉRATION DE NOURRITURE 
        void Ecosystem::SpawnFood(int count) { 
            for (int i = 0; i < count; ++i) { 
                if (mFoodSources.size() < 100) {  // Limite maximale de nourriture 
                    Vector2D position = GetRandomPosition(); 
                    mFoodSources.emplace_back(position, 25.0f); 
                } 
            }
        } 
        // SUPPRESSION DES ENTITÉS MORTES 
        void Ecosystem::RemoveDeadEntities() { 
            int initialCount = mEntities.size(); 
            mEntities.erase( 
                std::remove_if(mEntities.begin(), mEntities.end(), 
                    [](const std::unique_ptr<Entity>& entity) {  
                        return !entity->IsAlive();  
                    }), 
                mEntities.end() 
            ); 
            int removedCount = initialCount - mEntities.size(); 
            if (removedCount > 0) { 
                mStats.deathsToday += removedCount; 
            }
        } 
        // GESTION DE LA REPRODUCTION 
        void Ecosystem::HandleReproduction() { 
            std::vector<std::unique_ptr<Entity>> newEntities; 
            for (auto& entity : mEntities) { 
                if (entity->CanReproduce() && mEntities.size() < mMaxEntities) { 
                    auto baby = entity->Reproduce(); 
                    if (baby) { 
                        newEntities.push_back(std::move(baby)); 
                        mStats.birthsToday++; 
                    }
                } 
            } 
                        
            // Ajout des nouveaux entités 
            for (auto& newEntity : newEntities) { 
                mEntities.push_back(std::move(newEntity)); 
            }
        } 
        // 🍽 GESTION DE L'ALIMENTATION 
        void Ecosystem::HandleEating() { 
            // Ici on implémenterait la logique de recherche de nourriture 
            // Pour l'instant, gestion simplifiée 
            for (auto& entity : mEntities) { 
                if (entity->GetType() == EntityType::HERBIVORE) {
                    if(entity->GetEnergy() < 40.0f){
                        entity->position.x = entity->position.x + (entity->SeekFood(mFoodSources).x * 0.05f);
                        entity->position.y = entity->position.y + (entity->SeekFood(mFoodSources).y * 0.05f);
                        if((entity->SeekFood(mFoodSources).x * entity->SeekFood(mFoodSources).x + entity->SeekFood(mFoodSources).y * entity->SeekFood(mFoodSources).y))
                        entity->Eat(100.0f); 
                        }
                    }
                } 
            }
        } 
        // MISE À JOUR DES STATISTIQUES 
        void Ecosystem::UpdateStatistics() { 
            mStats.totalHerbivores = 0; 
            mStats.totalCarnivores = 0; 
            mStats.totalPlants = 0; 
            mStats.totalFood = mFoodSources.size(); 
            for (const auto& entity : mEntities) { 
                switch (entity->GetType()) { 
                    case EntityType::HERBIVORE: 
                        mStats.totalHerbivores++; 
                        break; 
                    case EntityType::CARNIVORE: 
                        mStats.totalCarnivores++; 
                        break; 
                    case EntityType::PLANT: 
                        mStats.totalPlants++; 
                        break; 
                } 
            }
        } 
        // CRÉATION D'ENTITÉ ALÉATOIRE 
        void Ecosystem::SpawnRandomEntity(EntityType type) { 
            if (mEntities.size() >= mMaxEntities) return; 
            Vector2D position = GetRandomPosition(); 
            std::string name; 
            switch (type) { 
                case EntityType::HERBIVORE: 
                    name = "Herbivore_" + std::to_string(mStats.totalHerbivores); 
                    break; 
                case EntityType::CARNIVORE: 
                    name = "Carnivore_" + std::to_string(mStats.totalCarnivores); 
                    break; 
                case EntityType::PLANT: 
                    name = "Plant_" + std::to_string(mStats.totalPlants); 
                    break; 
            }
            mEntities.push_back(std::make_unique<Entity>(type, position, name)); 
        } 
        // POSITION ALÉATOIRE 
        Vector2D Ecosystem::GetRandomPosition() const { 
            std::uniform_real_distribution<float> distX(0.0f, mWorldWidth); 
            std::uniform_real_distribution<float> distY(0.0f, mWorldHeight); 
            return Vector2D(distX(mRandomGenerator), distY(mRandomGenerator)); 
        } 
        // CROISSANCE DES PLANTES 
        void Ecosystem::HandlePlantGrowth(float deltaTime) { 
            // Occasionnellement, faire pousser de nouvelles plantes 
            std::uniform_real_distribution<float> chance(0.0f, 1.0f); 
            if (chance(mRandomGenerator) < 0.01f && mEntities.size() < mMaxEntities) { 
                SpawnRandomEntity(EntityType::PLANT); 
            }
        } 
        //RENDU 
        void Ecosystem::Render(SDL_Renderer* renderer) const { 
            // Rendu de la nourriture 
            for (const auto& food : mFoodSources) { 
                SDL_FRect rect = { 
                    food.position.x - 3.0f, 
                    food.position.y - 3.0f, 
                    6.0f, 
                    6.0f 
                };
                SDL_SetRenderDrawColor(renderer, food.color.r, food.color.g, food.color.b, food.color.a); 
                SDL_RenderFillRect(renderer, &rect); 
            }
            // Rendu des entités 
            for (const auto& entity : mEntities) { 
                entity->Render(renderer); 
            }
        } 
    } // namespace Core 
} // namespace Ecosystem
```

## src/Core/Entity.cpp ##
```cpp
#include "Core/Entity.h"
#include <cmath> 
#include <iostream> 
#include <algorithm> 
namespace Ecosystem { 
    namespace Core { 

        // 🏗 CONSTRUCTEUR PRINCIPAL 
        Entity::Entity(EntityType type, Vector2D pos, std::string entityName): mType(type), position(pos), name(entityName), mRandomGenerator(std::random_device{}())  // Initialisation du générateur alé
        { 
            // ��INITIALISATION SELON LE TYPE 
            switch(mType) { 
                case EntityType::HERBIVORE: 
                    mEnergy = 80.0f; 
                    mMaxEnergy = 150.0f; 
                    mMaxAge = 200; 
                    color = Color::Blue(); 
                    size = 8.0f; 
                break; 
                case EntityType::CARNIVORE: 
                    mEnergy = 100.0f; 
                    mMaxEnergy = 200.0f; 
                    mMaxAge = 150; 
                    color = Color::Red(); 
                    size = 12.0f; 
                    break; 
                case EntityType::PLANT: 
                    mEnergy = 50.0f; 
                    mMaxEnergy = 100.0f; 
                    mMaxAge = 300; 
                    color = Color::Green(); 
                    size = 6.0f; 
                break; 
            }
            mAge = 0; 
            mIsAlive = true; 
            mVelocity = GenerateRandomDirection(); 
            std::cout << "🌱Entité créée: " << name << " à (" << position.x << ", " << position.y << std::endl ;
        }
        // 🏗 CONSTRUCTEUR DE COPIE 
        Entity::Entity(const Entity& other) 
            : mType(other.mType), position(other.position), name(other.name + "_copy"), 
            mEnergy(other.mEnergy * 0.7f),  // Enfant a moins d'énergie 
            mMaxEnergy(other.mMaxEnergy), 
            mAge(0),  // Nouvelle entité, âge remis à 0 
            mMaxAge(other.mMaxAge), 
            mIsAlive(true), 
            mVelocity(other.mVelocity), 
            color(other.color), 
            size(other.size * 0.8f),  // Enfant plus petit 
            mRandomGenerator(std::random_device{}()) 
        { 
            std::cout << "👶Copie d'entité créée: " << name << std::endl; 
        } 
        
        // 🗑 DESTRUCTEUR 
        Entity::~Entity() { 
            std::cout << "💀Entité détruite: " << name << " (Âge: " << mAge << ")" << std::endl ;
        } 
        // ⚙ MISE À JOUR PRINCIPALE 
        void Entity::Update(float deltaTime) { 
            if (!mIsAlive) return; 
            // �� PROCESSUS DE VIE 
            ConsumeEnergy(deltaTime); 
            Age(deltaTime); 
            Move(deltaTime); 
            CheckVitality(); 
        }
        // ��MOUVEMENT 
        void Entity::Move(float deltaTime) { 
            if (mType == EntityType::PLANT) return; 
            // Les plantes ne bougent pas 
            // �� Comportement aléatoire occasionnel 

            std::uniform_real_distribution<float> chance(0.0f, 1.0f); 
            if (chance(mRandomGenerator) < 0.02f) { 
                mVelocity = GenerateRandomDirection(); 
            }

            // ��Application du mouvement 
            position = position + mVelocity * deltaTime * 30.0f; 
            //pour empecher la sortie de l ecrant
            position = StayInBounds(1200.0f, 800.0f);
            position = position + mVelocity * deltaTime * 20.0f;
            // �� Consommation d'énergie due au mouvement 
            mEnergy -= mVelocity.Distance(Vector2D(0, 0)) * deltaTime * 0.1f; 
        } 
        void Entity::Eat(float energy) { 
            mEnergy += energy; 
            if (mEnergy > mMaxEnergy) { 
                mEnergy = mMaxEnergy; 
            }
            std::cout << "🍽 " << name << " mange et gagne " << energy << " énergie" << std::endl ;
        } 
        Vector2D Entity::SeekFood(const std::vector<Food>& foodSources) const {
            Vector2D closestPos(0.0f, 0.0f);
            float closestDist = std::numeric_limits<float>::max();

            for(const Food& food : foodSources) {
                float dist = position.Distance(food.position);
                if (dist < closestDist) {
                    closestDist = dist;
                    closestPos = food.position;
                }
            }
            
            float length = std::sqrt(closestPos.x * closestPos.x + closestPos.y * closestPos.y);
            if(length > 0.0f){
                closestPos.x /= length;
                closestPos.y /= length;
            }
            return closestPos;
        }
        Vector2D Entity::StayInBounds(float worldWidth, float worldHeight){
            
            if (position.x < 0) position.x = 0;
            if (position.y < 0) position.y = 0;
            
            if (position.x > worldWidth) position.x = worldWidth;
            if (position.y > worldHeight) position.y = 0;

            return position;
        }
        // ��CONSOMMATION D'ÉNERGIE 
        void Entity::ConsumeEnergy(float deltaTime) { 
            float baseConsumption = 0.0f; 
            switch(mType) { 
                case EntityType::HERBIVORE: 
                    baseConsumption = 1.5f; 
                    break; 
                case EntityType::CARNIVORE: 
                    baseConsumption = 2.0f; 
                    break; 
                case EntityType::PLANT: 
                    baseConsumption = -0.5f;  // Les plantes génèrent de l'énergie ! 
                    break; 
            }
            mEnergy -= baseConsumption * deltaTime; 
        } 
        // ��VIEILLISSEMENT 
        void Entity::Age(float deltaTime) { 
            mAge += static_cast<int>(deltaTime * 10.0f);  // Accéléré pour la simulation 
        } 
        // ❤VÉRIFICATION DE LA SANTÉ 
        void Entity::CheckVitality() { 
            if (mEnergy <= 0.0f || mAge >= mMaxAge) { 
                mIsAlive = false; 
                std::cout << "💀" << name << " meurt - "; 
                if (mEnergy <= 0) std::cout << "Faim"; 
                else std::cout << "Vieillesse"; 
                std::cout << std::endl; 
            }
        } 

        // ��REPRODUCTION 
        bool Entity::CanReproduce() const { 
            return mIsAlive && mEnergy > mMaxEnergy * 0.8f && mAge > 20; 
        }

        std::unique_ptr<Entity> Entity::Reproduce() { 
            if (!CanReproduce()) return nullptr; 
            // ��Chance de reproduction 
            std::uniform_real_distribution<float> chance(0.0f, 1.0f); 
            if (chance(mRandomGenerator) < 0.3f) { 
            }
                mEnergy *= 0.6f;  // Coût énergétique de la reproduction 
                return std::make_unique<Entity>(*this);  // Utilise le constructeur de copi
                return nullptr; 
        } 

        // �� GÉNÉRATION DE DIRECTION ALÉATOIRE 
        Vector2D Entity::GenerateRandomDirection() { 
            std::uniform_real_distribution<float> dist(-1.0f, 1.0f); 
            return (dist(mRandomGenerator), dist(mRandomGenerator)); 
        } 

        // CALCUL DE LA COULEUR BASÉE SUR LÉTAT 
        Color Entity::CalculateColorBasedOnState() const { 
            float energyRatio = GetEnergyPercentage(); 
            Color baseColor = color; 
            // ��Rouge si faible énergie 
            if (energyRatio < 0.3f) { 
                baseColor.r = 255; 
                baseColor.g = static_cast<uint8_t>(baseColor.g * energyRatio); 
                baseColor.b = static_cast<uint8_t>(baseColor.b * energyRatio); 
            }
            return baseColor; 
        } 

        // ��RENDU GRAPHIQUE 
        void Entity::Render(SDL_Renderer* renderer) const { 
            if (!mIsAlive) return; 
            Color renderColor = CalculateColorBasedOnState(); 
            SDL_FRect rect = { 
                position.x - size / 2.0f, 
                position.y - size / 2.0f, 
                size, 
                size 
            }; 
            SDL_SetRenderDrawColor(renderer, renderColor.r, renderColor.g, renderColor.b, renderColor.a);
            SDL_RenderFillRect(renderer, &rect); 
            // ��Indicateur dénergie (barre de vie) 
            if (mType != EntityType::PLANT) { 
                float energyBarWidth = size * GetEnergyPercentage(); 
                SDL_FRect energyBar = { 
                    position.x - size / 2.0f, 
                    position.y - size / 2.0f - 3.0f, 
                    energyBarWidth, 
                    2.0f 
                };
                SDL_SetRenderDrawColor(renderer, 0, 255, 0, 255); 
                SDL_RenderFillRect(renderer, &energyBar); 
            }
            
        }
    };// namespace Core 
}; // namespace Ecosystem 
```

## src/Core/GameEngine.cpp ##
```cpp
#include "Core/GameEngine.h"
#include <iostream> 
#include <sstream> 
namespace Ecosystem { 
    namespace Core { 
        // 🏗 CONSTRUCTEUR 
        GameEngine::GameEngine(const std::string& title, float width, float height) 
            : mWindow(title, width, height),  
            mEcosystem(width, height, 500), 
            mIsRunning(false),  
            mIsPaused(false), 
            mTimeScale(1.0f), 
            mAccumulatedTime(0.0f) {} 
        // ⚙ INITIALISATION 
        bool GameEngine::Initialize() { 
            if (!mWindow.Initialize()) { 
                return false; 
            }
            mEcosystem.Initialize(20, 5, 30);  // 20 herbivores, 5 carnivores, 30 plantes 
            mIsRunning = true; 
            mLastUpdateTime =  std::chrono::high_resolution_clock::now(); 
            std::cout << "✅ Moteur de jeu initialisé" << std::endl; 
            return true; 
        } 
        // ��BOUCLE PRINCIPALE 
        void GameEngine::Run() { 
            std::cout << "🎯Démarrage de la boucle de jeu..." << std::endl; 
            while (mIsRunning) { 
                auto currentTime = std::chrono::high_resolution_clock::now(); 
                std::chrono::duration<float> elapsed = currentTime - mLastUpdateTime; 
                mLastUpdateTime = currentTime; 
                float deltaTime = elapsed.count(); 
                HandleEvents(); 
                if (!mIsPaused) { 
                    Update(deltaTime * mTimeScale); 
                } 
                Render(); 

                // Limitation à ~60 FPS 
                SDL_Delay(16); 
            } 

        }

        // �� FERMETURE 
        void GameEngine::Shutdown() { 
            mIsRunning = false; 
            std::cout << "🔄 Moteur de jeu arrêté" << std::endl; 
        } 
        // �� GESTION DES ÉVÉNEMENTS 
        void GameEngine::HandleEvents() { 

            SDL_Event event; 

            while (SDL_PollEvent(&event)) { 
                switch (event.type) { 
                    case SDL_EVENT_QUIT: 
                        mIsRunning = false; 
                    break; 
                    case SDL_EVENT_KEY_DOWN: 
                        HandleInput(event.key.key); 
                    break; 
                } 
            }
        } 


        // ⌨ GESTION DES TOUCHES 
        void GameEngine::HandleInput(SDL_Keycode key) { 
            switch (key) { 
                case SDLK_ESCAPE: 
                    mIsRunning = false; 
                    break; 
                case SDLK_SPACE: 
                    mIsPaused = !mIsPaused; 
                    std::cout << (mIsPaused ? "⏸Simulation en pause" : "▶ Simulation reproduction")<< std::endl ;
                    break; 
                case SDLK_R: 
                    mEcosystem.Initialize(20, 5, 30); 
                    std::cout << "🔄 Simulation réinitialisée" << std::endl; 
                    break; 
                case SDLK_F: 
                    mEcosystem.SpawnFood(10); 
                    std::cout << "🍎 Nourriture ajoutée" << std::endl; 
                    break; 
                case SDLK_UP: 
                    mTimeScale *= 1.5f; 
                    std::cout << "⏩ Vitesse: " << mTimeScale << "x" << std::endl; 
                    break; 
                case SDLK_DOWN: 
                    mTimeScale /= 1.5f; 
                    std::cout << " ⏪ Vitesse: " << mTimeScale << "x" << std::endl; 
                    break; 
            }
        } 

        // �� MISE À JOUR 
        void GameEngine::Update(float deltaTime) { 
            mEcosystem.Update(deltaTime); 
            // Affichage occasionnel des statistiques 
            static float statsTimer = 0.0f; 
            statsTimer += deltaTime; 
            if (statsTimer >= 2.0f) { 
                auto stats = mEcosystem.GetStatistics(); 
                std::cout << "📊 Stats - Herbivores: " << stats.totalHerbivores  
                        << ", Carnivores: " << stats.totalCarnivores 
                        << ", Plantes: " << stats.totalPlants 
                        << ", Naissances: " << stats.birthsToday 
                        << ", Morts: " << stats.deathsToday << std::endl; 
                statsTimer = 0.0f; 
            }
        } 


        // �� RENDU 
        void GameEngine::Render() { 
            mWindow.Clear(); 
            // Rendu de l'écosystème 
            mEcosystem.Render(mWindow.GetRenderer()); 
            // Ici on ajouterait l'interface utilisateur 
            RenderUI(); 
            mWindow.Present(); 
        } 


        // �� INTERFACE UTILISATEUR 
        void GameEngine::RenderUI() { 
            // Pour l'instant, interface texte dans la console 
            // Une vraie interface graphique serait implémentée ici 
        } 
    } // namespace Core
} // namespace Ecosystem
```

## src/Graphics/Renderer.cpp ##
```cpp
#include <SDL3/SDL.h> 
#include <string> 
#include "Core/Structs.h" 
namespace Ecosystem { 
namespace Graphics { 
class Window { 
private: 
    // RESSOURCES SDL 
    SDL_Window* mWindow; 
    SDL_Renderer* mRenderer; 
    float mWidth; 
    float mHeight; 
    bool mIsInitialized; 
    std::string mTitle; 
public: 
    // 🏗 CONSTRUCTEUR/DESTRUCTEUR 
    Window(const std::string& title, float width, float height); 
    ~Window(); 
    //INITIALISATION 
    bool Initialize(); 
    void Shutdown(); 
     
    //RENDU 
    void Clear(const Core::Color& color = Core::Color(30, 30, 30)); 
    void Present(); 
     
    //GETTERS 
    SDL_Renderer* GetRenderer() const { return mRenderer; } 
    bool IsInitialized() const { return mIsInitialized; } 
    float GetWidth() const { return mWidth; } 
    float GetHeight() const { return mHeight; } 
    std::string GetTitle() const { return mTitle; } 
}; 
} // namespace Graphics 
} // namespace Ecosystem 
```

## src/Graphics/Window.cpp ##
```cpp
#include "Graphics/Window.h" 
#include <iostream> 
namespace Ecosystem { 
namespace Graphics { 
// 🏗 CONSTRUCTEUR 
Window::Window(const std::string& title, float width, float height): mTitle(title), mWidth(width), mHeight(height), mWindow(nullptr), mRenderer(nullptr), mIsInitialized(false) {} 
// 🗑 DESTRUCTEUR 
Window::~Window() { 
    Shutdown(); 
} 
// ⚙ INITIALISATION 
bool Window::Initialize() { 
    //if (SDL_Init(SDL_INIT_VIDEO) != 0) { 
       // std::cerr << "❌Erreur SDL_Init: " << SDL_GetError() << std::endl; 
       // return false; 
   // }
    mWindow = SDL_CreateWindow(mTitle.c_str(), static_cast<int>(mWidth), static_cast<int>(mHeight),0); 
    if (!mWindow) { 
        std::cerr << "❌ Erreur création fenêtre: " << SDL_GetError() << std::endl;
        SDL_Quit(); 
        return false; 
    }
    mRenderer = SDL_CreateRenderer(mWindow, NULL); 
    if (!mRenderer) { 
        std::cerr << "❌Erreur création renderer: " << SDL_GetError() << std::endl ;
        SDL_DestroyWindow(mWindow); 
        SDL_Quit(); 
        return false; 
    }
    mIsInitialized = true; 
    std::cout << "✅Fenêtre initialisée: " << mTitle << " (" << mWidth << "x" << mHeight << std::endl ;
    return true; 
} 
// �� FERMETURE 
void Window::Shutdown() { 
    if (mRenderer) { 
        SDL_DestroyRenderer(mRenderer); 
        mRenderer = nullptr; 
    }
    if (mWindow) { 
        SDL_DestroyWindow(mWindow); 
        mWindow = nullptr; 
    }
    SDL_Quit(); 
    mIsInitialized = false; 
    std::cout << "🔄 Fenêtre fermée" << std::endl; 
} 
// �� NETTOYAGE DE LÉCRAN 
void Window::Clear(const Core::Color& color) { 
    if (mRenderer) { 
        SDL_SetRenderDrawColor(mRenderer, color.r, color.g, color.b, color.a); 
        SDL_RenderClear(mRenderer); 
    }
 } 
// �� AFFICHAGE 
void Window::Present() { 
    if (mRenderer) { 
        SDL_RenderPresent(mRenderer); 
    }
 } 
} // namespace Graphics 
} // namespace Ecosystem 
```
## Main.cpp ##
```cpp
#include "Core/GameEngine.h" 
#include <iostream> 
#include <cstdlib> 
#include <ctime> 
int main(int argc, char* argv[]) { 
    // �� Initialisation de laléatoire 
    std::srand(static_cast<unsigned int>(std::time(nullptr))); 
     
    std::cout << "🎮 Démarrage du Simulateur d'Écosystème" << std::endl; 
    std::cout << "=======================================" << std::endl; 
     
    // 🏗 Création du moteur de jeu 
    Ecosystem::Core::GameEngine engine("Simulateur d'Écosystème Intelligent", 1200.0f, 800.0f);
     
    // ⚙Initialisation 
    if (!engine.Initialize()) { 
        std::cerr << "❌ Erreur: Impossible d'initialiser le moteur de jeu" << std::endl ;
        return -1; 
    }
     
    std::cout << "✅ Moteur initialisé avec succès" << std::endl; 
    std::cout << "🎯 Lancement de la simulation..." << std::endl; 
    std::cout << "=== CONTRÔLES ===" << std::endl; 
    std::cout << "ESPACE: Pause/Reprise" << std::endl; 
    std::cout << "R: Reset simulation" << std::endl; 
    std::cout << "F: Ajouter nourriture" << std::endl; 
    std::cout << "FLÈCHES: Vitesse simulation" << std::endl; 
    std::cout << "ÉCHAP: Quitter" << std::endl; 
     
    // �� Boucle principale 
    engine.Run(); 
     
    // ��Arrêt propre 
    engine.Shutdown(); 
     
    std::cout << "👋 Simulation terminée. Au revoir !" << std::endl; 
    return 0; 
} 
```
## 👌👌 CONCEPT APPRIS DANS CE PROJET ##
- La programmation orientee objet(classes, methodes)
- Gestion de vector 2D
- Boucle, condition, distance
- Organisation d un proget en c++

## 👌👌 CONCEPT MATHEMATIQUE APPRIS ##
- Algebre 2D
- Calcul vectoriel
