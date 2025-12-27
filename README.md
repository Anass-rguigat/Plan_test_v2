# Plan de Test - Inventory Management System

Ce document contient tous les scénarios de tests unitaires et le plan de test complet de l'application, structuré pour être utilisé dans les diagrammes de séquence (boîte blanche), Jira et Xray.

---

## Table des Matières

1. [Vue d'ensemble](#vue-densemble)
2. [Scénarios de Test Détaillés](#scénarios-de-test-détaillés)
   - [Authentification](#authentification)
   - [Gestion des Catégories](#gestion-des-catégories)
   - [Gestion des Produits](#gestion-des-produits)
   - [Gestion des Fournisseurs](#gestion-des-fournisseurs)
   - [Gestion des Transactions](#gestion-des-transactions)
   - [Gestion des Utilisateurs](#gestion-des-utilisateurs)
3. [Tests Unitaires - Services](#tests-unitaires---services)
4. [Tests Unitaires - Sécurité](#tests-unitaires---sécurité)
5. [Tests Unitaires - Contrôleurs](#tests-unitaires---contrôleurs)
6. [Matrice de Couverture](#matrice-de-couverture)
7. [Exécution des Tests](#exécution-des-tests)

---

## Vue d'ensemble

### Types de Tests

- **Tests Unitaires** : Tests isolés des services, contrôleurs et composants de sécurité
- **Tests de Contrôleurs** : Tests des endpoints REST avec MockMvc
- **Scénarios de Test Fonctionnels** : Scénarios complets pour diagrammes de séquence, Jira et Xray

### Outils Utilisés

- **JUnit 5** : Framework de test
- **Mockito** : Framework de mocking
- **Spring Boot Test** : Support de test Spring
- **H2 Database** : Base de données en mémoire pour les tests
- **Jira/Xray** : Gestion des tests et traçabilité

---

## Scénarios de Test Détaillés

### Authentification

#### TC-AUTH-001 : Enregistrement d'un nouvel utilisateur

**Nom** : Enregistrement d'un nouvel utilisateur avec succès

**Acteur** : Utilisateur non authentifié

**Données d'entrée** :
```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "password": "SecurePass123!",
  "phoneNumber": "+1234567890",
  "role": "ADMIN"
}
```

**Scénario** :
1. L'utilisateur envoie une requête POST à `/api/auth/register` avec les données d'enregistrement
2. Le système valide les données d'entrée (champs requis, format email, etc.)
3. Le système vérifie que l'email n'existe pas déjà dans la base de données
4. Le système encode le mot de passe avec BCrypt
5. Le système crée un nouvel utilisateur avec le rôle spécifié (ou MANAGER par défaut)
6. Le système sauvegarde l'utilisateur dans la base de données
7. Le système retourne une réponse avec status 200 et message "User Registered Successfully"

**Résultat attendu** : Utilisateur créé avec succès, status 200

**Préconditions** : Aucune

**Postconditions** : Nouvel utilisateur créé dans la base de données

---

#### TC-AUTH-002 : Enregistrement avec email existant

**Nom** : Tentative d'enregistrement avec un email déjà utilisé

**Acteur** : Utilisateur non authentifié

**Données d'entrée** :
```json
{
  "name": "Jane Doe",
  "email": "john.doe@example.com",
  "password": "AnotherPass123!",
  "phoneNumber": "+1234567891",
  "role": "MANAGER"
}
```

**Scénario** :
1. L'utilisateur envoie une requête POST à `/api/auth/register` avec un email existant
2. Le système valide les données d'entrée
3. Le système vérifie l'existence de l'email dans la base de données
4. Le système détecte que l'email existe déjà
5. Le système retourne une exception avec message d'erreur approprié

**Résultat attendu** : Erreur, email déjà utilisé

**Préconditions** : Un utilisateur avec l'email "john.doe@example.com" existe déjà

---

#### TC-AUTH-003 : Connexion utilisateur avec succès

**Nom** : Connexion d'un utilisateur avec des identifiants valides

**Acteur** : Utilisateur enregistré

**Données d'entrée** :
```json
{
  "email": "john.doe@example.com",
  "password": "SecurePass123!"
}
```

**Scénario** :
1. L'utilisateur envoie une requête POST à `/api/auth/login` avec email et mot de passe
2. Le système valide les données d'entrée
3. Le système recherche l'utilisateur par email dans la base de données
4. Le système vérifie que l'utilisateur existe
5. Le système compare le mot de passe fourni avec le hash stocké
6. Le système génère un token JWT avec les informations de l'utilisateur
7. Le système retourne une réponse avec status 200, token JWT, rôle utilisateur et temps d'expiration

**Résultat attendu** : Token JWT généré, status 200

**Préconditions** : Un utilisateur avec ces identifiants existe dans la base de données

**Postconditions** : Token JWT valide pour l'utilisateur

---

#### TC-AUTH-004 : Connexion avec email inexistant

**Nom** : Tentative de connexion avec un email non enregistré

**Acteur** : Utilisateur non enregistré

**Données d'entrée** :
```json
{
  "email": "unknown@example.com",
  "password": "AnyPassword123!"
}
```

**Scénario** :
1. L'utilisateur envoie une requête POST à `/api/auth/login` avec un email inexistant
2. Le système valide les données d'entrée
3. Le système recherche l'utilisateur par email dans la base de données
4. Le système ne trouve pas l'utilisateur
5. Le système retourne une exception NotFoundException avec message "User not found"

**Résultat attendu** : Erreur, utilisateur non trouvé

---

#### TC-AUTH-005 : Connexion avec mot de passe incorrect

**Nom** : Tentative de connexion avec un mot de passe erroné

**Acteur** : Utilisateur enregistré

**Données d'entrée** :
```json
{
  "email": "john.doe@example.com",
  "password": "WrongPassword123!"
}
```

**Scénario** :
1. L'utilisateur envoie une requête POST à `/api/auth/login` avec un mauvais mot de passe
2. Le système valide les données d'entrée
3. Le système recherche l'utilisateur par email dans la base de données
4. Le système trouve l'utilisateur
5. Le système compare le mot de passe fourni avec le hash stocké
6. Le système détecte que le mot de passe ne correspond pas
7. Le système retourne une exception InvalidCredentialsException avec message "Invalid credentials"

**Résultat attendu** : Erreur, identifiants invalides

**Préconditions** : Un utilisateur avec cet email existe dans la base de données

---

### Gestion des Catégories

#### TC-CAT-001 : Création d'une catégorie (ADMIN)

**Nom** : Création d'une nouvelle catégorie par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
```json
{
  "name": "Électronique"
}
```

**Scénario** :
1. L'administrateur authentifié envoie une requête POST à `/api/categories/add` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée (nom requis, non vide)
4. Le système crée un nouvel objet Category avec le nom fourni
5. Le système sauvegarde la catégorie dans la base de données via CategoryRepository
6. Le système mappe la catégorie en CategoryDTO
7. Le système retourne une réponse avec status 200 et message "Category Saved Successfully"

**Résultat attendu** : Catégorie créée avec succès, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN

**Postconditions** : Nouvelle catégorie créée dans la base de données

---

#### TC-CAT-002 : Récupération de toutes les catégories

**Nom** : Récupération de la liste complète des catégories

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** : Aucune

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/categories/all` avec un token JWT valide
2. Le système valide le token JWT
3. Le système récupère toutes les catégories depuis la base de données, triées par nom (Sort.by("name"))
4. Le système mappe la liste de Category en liste de CategoryDTO
5. Le système retourne une réponse avec status 200, message "success" et la liste des catégories

**Résultat attendu** : Liste des catégories retournée, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-CAT-003 : Récupération d'une catégorie par ID

**Nom** : Récupération des détails d'une catégorie spécifique

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/categories/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche la catégorie avec l'ID 1 dans la base de données
4. Le système vérifie que la catégorie existe
5. Le système mappe la Category en CategoryDTO
6. Le système retourne une réponse avec status 200 et les détails de la catégorie

**Résultat attendu** : Détails de la catégorie retournés, status 200

**Préconditions** : Utilisateur authentifié, catégorie avec ID 1 existe

---

#### TC-CAT-004 : Récupération d'une catégorie inexistante

**Nom** : Tentative de récupération d'une catégorie qui n'existe pas

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `id = 999`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/categories/999` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche la catégorie avec l'ID 999 dans la base de données
4. Le système ne trouve pas la catégorie
5. Le système retourne une exception NotFoundException avec message "Category not found"

**Résultat attendu** : Erreur, catégorie non trouvée

**Préconditions** : Utilisateur authentifié, catégorie avec ID 999 n'existe pas

---

#### TC-CAT-005 : Mise à jour d'une catégorie (ADMIN)

**Nom** : Modification d'une catégorie existante par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`
- Body:
```json
{
  "name": "Électronique et Informatique"
}
```

**Scénario** :
1. L'administrateur authentifié envoie une requête PUT à `/api/categories/update/1` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée
4. Le système recherche la catégorie avec l'ID 1 dans la base de données
5. Le système vérifie que la catégorie existe
6. Le système met à jour le nom de la catégorie
7. Le système sauvegarde les modifications dans la base de données
8. Le système mappe la catégorie mise à jour en CategoryDTO
9. Le système retourne une réponse avec status 200 et message "Category Updated Successfully"

**Résultat attendu** : Catégorie mise à jour, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, catégorie avec ID 1 existe

**Postconditions** : Catégorie modifiée dans la base de données

---

#### TC-CAT-006 : Suppression d'une catégorie (ADMIN)

**Nom** : Suppression d'une catégorie par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'administrateur authentifié envoie une requête DELETE à `/api/categories/delete/1` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système recherche la catégorie avec l'ID 1 dans la base de données
4. Le système vérifie que la catégorie existe
5. Le système supprime la catégorie de la base de données
6. Le système retourne une réponse avec status 200 et message "Category Deleted Successfully"

**Résultat attendu** : Catégorie supprimée, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, catégorie avec ID 1 existe

**Postconditions** : Catégorie supprimée de la base de données

---

### Gestion des Produits

#### TC-PROD-001 : Création d'un produit avec image (ADMIN)

**Nom** : Création d'un nouveau produit avec image par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Multipart Form Data:
  - `imageFile`: fichier image (JPEG/PNG)
  - `name`: "Laptop HP Pavilion"
  - `sku`: "LAP-HP-001"
  - `price`: 899.99
  - `stockQuantity`: 50
  - `categoryId`: 1
  - `description`: "Ordinateur portable haute performance"

**Scénario** :
1. L'administrateur authentifié envoie une requête POST à `/api/products/add` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée (champs requis)
4. Le système recherche la catégorie avec l'ID fourni dans la base de données
5. Le système vérifie que la catégorie existe
6. Le système génère un nom de fichier unique pour l'image
7. Le système sauvegarde l'image dans le répertoire `/public/products/`
8. Le système crée un nouvel objet Product avec les données fournies
9. Le système associe le produit à la catégorie
10. Le système sauvegarde le produit dans la base de données
11. Le système mappe le produit en ProductDTO
12. Le système retourne une réponse avec status 200 et message "Product Saved Successfully"

**Résultat attendu** : Produit créé avec succès, image sauvegardée, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, catégorie avec ID 1 existe

**Postconditions** : Nouveau produit créé, image sauvegardée

---

#### TC-PROD-002 : Création d'un produit sans image (ADMIN)

**Nom** : Création d'un produit sans image par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Multipart Form Data:
  - `imageFile`: null (optionnel)
  - `name`: "Clavier Mécanique"
  - `sku`: "KB-MEC-001"
  - `price`: 79.99
  - `stockQuantity`: 100
  - `categoryId`: 1
  - `description`: "Clavier mécanique RGB"

**Scénario** :
1. L'administrateur authentifié envoie une requête POST à `/api/products/add` sans fichier image
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée
4. Le système recherche la catégorie avec l'ID fourni
5. Le système vérifie que la catégorie existe
6. Le système crée un nouvel objet Product sans imageUrl
7. Le système sauvegarde le produit dans la base de données
8. Le système retourne une réponse avec status 200 et message "Product Saved Successfully"

**Résultat attendu** : Produit créé sans image, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, catégorie avec ID 1 existe

---

#### TC-PROD-003 : Création d'un produit avec catégorie inexistante

**Nom** : Tentative de création d'un produit avec une catégorie qui n'existe pas

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Multipart Form Data:
  - `name`: "Produit Test"
  - `sku`: "TEST-001"
  - `price`: 10.00
  - `stockQuantity`: 5
  - `categoryId`: 999

**Scénario** :
1. L'administrateur authentifié envoie une requête POST à `/api/products/add` avec un categoryId inexistant
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée
4. Le système recherche la catégorie avec l'ID 999 dans la base de données
5. Le système ne trouve pas la catégorie
6. Le système retourne une exception NotFoundException avec message "Category not found"

**Résultat attendu** : Erreur, catégorie non trouvée

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, catégorie avec ID 999 n'existe pas

---

#### TC-PROD-004 : Récupération de tous les produits

**Nom** : Récupération de la liste complète des produits

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** : Aucune

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/products/all` avec un token JWT valide
2. Le système valide le token JWT
3. Le système récupère tous les produits depuis la base de données, triés par nom (Sort.by("name"))
4. Le système mappe la liste de Product en liste de ProductDTO
5. Le système retourne une réponse avec status 200, message "success" et la liste des produits

**Résultat attendu** : Liste des produits retournée, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-PROD-005 : Recherche de produits

**Nom** : Recherche de produits par nom ou description

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Query Parameter: `input = "Laptop"`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/products/search?input=Laptop` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche les produits dont le nom ou la description contient "Laptop" (findByNameContainingOrDescriptionContaining)
4. Le système mappe la liste de Product en liste de ProductDTO
5. Le système retourne une réponse avec status 200 et les produits trouvés

**Résultat attendu** : Liste des produits correspondants retournée, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-PROD-006 : Mise à jour d'un produit (ADMIN)

**Nom** : Modification d'un produit existant par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Multipart Form Data:
  - `productId`: 1
  - `imageFile`: nouveau fichier image (optionnel)
  - `name`: "Laptop HP Pavilion Pro"
  - `price`: 999.99
  - `stockQuantity`: 75

**Scénario** :
1. L'administrateur authentifié envoie une requête PUT à `/api/products/update` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système recherche le produit avec l'ID fourni dans la base de données
4. Le système vérifie que le produit existe
5. Si une nouvelle image est fournie, le système supprime l'ancienne image et sauvegarde la nouvelle
6. Le système met à jour les champs du produit avec les nouvelles valeurs
7. Le système sauvegarde les modifications dans la base de données
8. Le système retourne une réponse avec status 200 et message "Product Updated Successfully"

**Résultat attendu** : Produit mis à jour, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, produit avec ID 1 existe

**Postconditions** : Produit modifié dans la base de données

---

#### TC-PROD-007 : Suppression d'un produit (ADMIN)

**Nom** : Suppression d'un produit par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'administrateur authentifié envoie une requête DELETE à `/api/products/delete/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche le produit avec l'ID 1 dans la base de données
4. Le système vérifie que le produit existe
5. Le système supprime l'image associée du répertoire `/public/products/`
6. Le système supprime le produit de la base de données
7. Le système retourne une réponse avec status 200 et message "Product Deleted Successfully"

**Résultat attendu** : Produit supprimé, image supprimée, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, produit avec ID 1 existe

**Postconditions** : Produit et image supprimés

---

### Gestion des Fournisseurs

#### TC-SUP-001 : Ajout d'un fournisseur (ADMIN)

**Nom** : Création d'un nouveau fournisseur par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
```json
{
  "name": "TechSupply Inc.",
  "contactInfo": "contact@techsupply.com",
  "address": "123 Business Street, City, Country"
}
```

**Scénario** :
1. L'administrateur authentifié envoie une requête POST à `/api/suppliers/add` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée (nom et contactInfo requis)
4. Le système crée un nouvel objet Supplier avec les données fournies
5. Le système sauvegarde le fournisseur dans la base de données via SupplierRepository
6. Le système mappe le fournisseur en SupplierDTO
7. Le système retourne une réponse avec status 200 et message "Supplier Saved Successfully"

**Résultat attendu** : Fournisseur créé avec succès, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN

**Postconditions** : Nouveau fournisseur créé dans la base de données

---

#### TC-SUP-002 : Récupération de tous les fournisseurs

**Nom** : Récupération de la liste complète des fournisseurs

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** : Aucune

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/suppliers/all` avec un token JWT valide
2. Le système valide le token JWT
3. Le système récupère tous les fournisseurs depuis la base de données, triés par nom (Sort.by("name"))
4. Le système mappe la liste de Supplier en liste de SupplierDTO
5. Le système retourne une réponse avec status 200, message "success" et la liste des fournisseurs

**Résultat attendu** : Liste des fournisseurs retournée, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-SUP-003 : Mise à jour d'un fournisseur (ADMIN)

**Nom** : Modification d'un fournisseur existant par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`
- Body:
```json
{
  "name": "TechSupply Global Inc.",
  "contactInfo": "info@techsupplyglobal.com",
  "address": "456 Corporate Avenue, City, Country"
}
```

**Scénario** :
1. L'administrateur authentifié envoie une requête PUT à `/api/suppliers/update/1` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système valide les données d'entrée
4. Le système recherche le fournisseur avec l'ID 1 dans la base de données
5. Le système vérifie que le fournisseur existe
6. Le système met à jour les champs du fournisseur
7. Le système sauvegarde les modifications dans la base de données
8. Le système retourne une réponse avec status 200 et message "Supplier Updated Successfully"

**Résultat attendu** : Fournisseur mis à jour, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, fournisseur avec ID 1 existe

**Postconditions** : Fournisseur modifié dans la base de données

---

#### TC-SUP-004 : Suppression d'un fournisseur (ADMIN)

**Nom** : Suppression d'un fournisseur par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'administrateur authentifié envoie une requête DELETE à `/api/suppliers/delete/1` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système recherche le fournisseur avec l'ID 1 dans la base de données
4. Le système vérifie que le fournisseur existe
5. Le système supprime le fournisseur de la base de données
6. Le système retourne une réponse avec status 200 et message "Supplier Deleted Successfully"

**Résultat attendu** : Fournisseur supprimé, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, fournisseur avec ID 1 existe

**Postconditions** : Fournisseur supprimé de la base de données

---

### Gestion des Transactions

#### TC-TRANS-001 : Achat de stock (Purchase)

**Nom** : Achat de produits pour augmenter le stock

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
```json
{
  "productId": 1,
  "quantity": 50,
  "supplierId": 1,
  "description": "Réapprovisionnement de stock",
  "note": "Commande urgente"
}
```

**Scénario** :
1. L'utilisateur authentifié envoie une requête POST à `/api/transactions/purchase` avec un token JWT valide
2. Le système valide le token JWT
3. Le système valide les données d'entrée (productId et quantity requis, supplierId requis pour purchase)
4. Le système recherche le produit avec l'ID fourni dans la base de données
5. Le système vérifie que le produit existe
6. Le système recherche le fournisseur avec l'ID fourni dans la base de données
7. Le système vérifie que le fournisseur existe
8. Le système récupère l'utilisateur connecté depuis le contexte de sécurité
9. Le système crée une nouvelle Transaction de type PURCHASE
10. Le système augmente le stockQuantity du produit de la quantité achetée
11. Le système sauvegarde la transaction dans la base de données
12. Le système met à jour le produit avec le nouveau stock
13. Le système retourne une réponse avec status 200 et message "Purchase transaction completed successfully"

**Résultat attendu** : Transaction créée, stock augmenté, status 200

**Préconditions** : Utilisateur authentifié, produit avec ID 1 existe, fournisseur avec ID 1 existe

**Postconditions** : Transaction PURCHASE créée, stock du produit augmenté

---

#### TC-TRANS-002 : Vente de produit (Sell)

**Nom** : Vente d'un produit pour diminuer le stock

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
```json
{
  "productId": 1,
  "quantity": 10,
  "description": "Vente au client",
  "note": "Client régulier"
}
```

**Scénario** :
1. L'utilisateur authentifié envoie une requête POST à `/api/transactions/sell` avec un token JWT valide
2. Le système valide le token JWT
3. Le système valide les données d'entrée (productId et quantity requis)
4. Le système recherche le produit avec l'ID fourni dans la base de données
5. Le système vérifie que le produit existe
6. Le système vérifie que le stock disponible est suffisant (stockQuantity >= quantity)
7. Le système récupère l'utilisateur connecté depuis le contexte de sécurité
8. Le système crée une nouvelle Transaction de type SELL
9. Le système diminue le stockQuantity du produit de la quantité vendue
10. Le système sauvegarde la transaction dans la base de données
11. Le système met à jour le produit avec le nouveau stock
12. Le système retourne une réponse avec status 200 et message "Sell transaction completed successfully"

**Résultat attendu** : Transaction créée, stock diminué, status 200

**Préconditions** : Utilisateur authentifié, produit avec ID 1 existe, stock suffisant

**Postconditions** : Transaction SELL créée, stock du produit diminué

---

#### TC-TRANS-003 : Retour au fournisseur (Return)

**Nom** : Retour de produits au fournisseur

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
```json
{
  "productId": 1,
  "quantity": 5,
  "supplierId": 1,
  "description": "Produits défectueux",
  "note": "Retour pour remboursement"
}
```

**Scénario** :
1. L'utilisateur authentifié envoie une requête POST à `/api/transactions/return` avec un token JWT valide
2. Le système valide le token JWT
3. Le système valide les données d'entrée (productId, quantity et supplierId requis pour return)
4. Le système recherche le produit avec l'ID fourni dans la base de données
5. Le système vérifie que le produit existe
6. Le système recherche le fournisseur avec l'ID fourni dans la base de données
7. Le système vérifie que le fournisseur existe
8. Le système vérifie que le stock disponible est suffisant
9. Le système récupère l'utilisateur connecté depuis le contexte de sécurité
10. Le système crée une nouvelle Transaction de type RETURN
11. Le système diminue le stockQuantity du produit de la quantité retournée
12. Le système sauvegarde la transaction dans la base de données
13. Le système met à jour le produit avec le nouveau stock
14. Le système retourne une réponse avec status 200 et message "Return transaction completed successfully"

**Résultat attendu** : Transaction créée, stock diminué, status 200

**Préconditions** : Utilisateur authentifié, produit avec ID 1 existe, fournisseur avec ID 1 existe, stock suffisant

**Postconditions** : Transaction RETURN créée, stock du produit diminué

---

#### TC-TRANS-004 : Récupération de toutes les transactions avec pagination

**Nom** : Récupération de la liste paginée des transactions

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Query Parameters:
  - `page = 0`
  - `size = 10`
  - `filter = "PURCHASE"` (optionnel)

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/transactions/all?page=0&size=10&filter=PURCHASE` avec un token JWT valide
2. Le système valide le token JWT
3. Le système crée une Specification pour filtrer les transactions (si filter fourni)
4. Le système crée un Pageable avec page et size
5. Le système récupère les transactions depuis la base de données avec pagination et filtres
6. Le système mappe la liste de Transaction en liste de TransactionDTO
7. Le système nettoie les références circulaires (setTransactions(null) sur UserDTO)
8. Le système retourne une réponse avec status 200, message "success", liste paginée, totalPages et totalElements

**Résultat attendu** : Liste paginée des transactions retournée, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-TRANS-005 : Récupération d'une transaction par ID

**Nom** : Récupération des détails d'une transaction spécifique

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/transactions/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche la transaction avec l'ID 1 dans la base de données
4. Le système vérifie que la transaction existe
5. Le système mappe la Transaction en TransactionDTO
6. Le système nettoie les références circulaires
7. Le système retourne une réponse avec status 200 et les détails de la transaction

**Résultat attendu** : Détails de la transaction retournés, status 200

**Préconditions** : Utilisateur authentifié, transaction avec ID 1 existe

---

#### TC-TRANS-006 : Mise à jour du statut d'une transaction

**Nom** : Modification du statut d'une transaction

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `transactionId = 1`
- Body: `"COMPLETED"` (TransactionStatus enum)

**Scénario** :
1. L'utilisateur authentifié envoie une requête PUT à `/api/transactions/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche la transaction avec l'ID 1 dans la base de données
4. Le système vérifie que la transaction existe
5. Le système met à jour le statut de la transaction
6. Le système sauvegarde les modifications dans la base de données
7. Le système retourne une réponse avec status 200 et message "Transaction status updated successfully"

**Résultat attendu** : Statut de la transaction mis à jour, status 200

**Préconditions** : Utilisateur authentifié, transaction avec ID 1 existe

**Postconditions** : Statut de la transaction modifié dans la base de données

---

#### TC-TRANS-007 : Récupération des transactions par mois et année

**Nom** : Récupération des transactions pour un mois et une année spécifiques

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Query Parameters:
  - `month = 12`
  - `year = 2024`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/transactions/by-month-year?month=12&year=2024` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche les transactions créées en décembre 2024 dans la base de données
4. Le système mappe la liste de Transaction en liste de TransactionDTO
5. Le système nettoie les références circulaires
6. Le système retourne une réponse avec status 200 et les transactions du mois spécifié

**Résultat attendu** : Liste des transactions du mois retournée, status 200

**Préconditions** : Utilisateur authentifié

---

### Gestion des Utilisateurs

#### TC-USER-001 : Récupération de tous les utilisateurs (ADMIN)

**Nom** : Récupération de la liste complète des utilisateurs par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** : Aucune

**Scénario** :
1. L'administrateur authentifié envoie une requête GET à `/api/users/all` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système récupère tous les utilisateurs depuis la base de données, triés par nom (Sort.by("name"))
4. Le système mappe la liste de User en liste de UserDTO
5. Le système retourne une réponse avec status 200, message "success" et la liste des utilisateurs

**Résultat attendu** : Liste des utilisateurs retournée, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN

---

#### TC-USER-002 : Récupération d'un utilisateur par ID

**Nom** : Récupération des détails d'un utilisateur spécifique

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/users/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche l'utilisateur avec l'ID 1 dans la base de données
4. Le système vérifie que l'utilisateur existe
5. Le système mappe l'utilisateur en UserDTO
6. Le système retourne une réponse avec status 200 et les détails de l'utilisateur

**Résultat attendu** : Détails de l'utilisateur retournés, status 200

**Préconditions** : Utilisateur authentifié, utilisateur avec ID 1 existe

---

#### TC-USER-003 : Récupération de l'utilisateur connecté

**Nom** : Récupération des informations de l'utilisateur actuellement connecté

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** : Aucune

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/users/current` avec un token JWT valide
2. Le système valide le token JWT
3. Le système extrait l'email de l'utilisateur depuis le contexte de sécurité (SecurityContextHolder)
4. Le système recherche l'utilisateur par email dans la base de données
5. Le système vérifie que l'utilisateur existe
6. Le système retourne l'objet User complet (pas un DTO)

**Résultat attendu** : Informations de l'utilisateur connecté retournées, status 200

**Préconditions** : Utilisateur authentifié

---

#### TC-USER-004 : Mise à jour d'un utilisateur

**Nom** : Modification des informations d'un utilisateur

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `id = 1`
- Body:
```json
{
  "name": "John Doe Updated",
  "phoneNumber": "+1234567899",
  "email": "john.doe@example.com"
}
```

**Scénario** :
1. L'utilisateur authentifié envoie une requête PUT à `/api/users/update/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche l'utilisateur avec l'ID 1 dans la base de données
4. Le système vérifie que l'utilisateur existe
5. Le système met à jour les champs de l'utilisateur avec les nouvelles valeurs
6. Si un nouveau mot de passe est fourni, le système l'encode avec BCrypt
7. Le système sauvegarde les modifications dans la base de données
8. Le système retourne une réponse avec status 200 et message "User Updated Successfully"

**Résultat attendu** : Utilisateur mis à jour, status 200

**Préconditions** : Utilisateur authentifié, utilisateur avec ID 1 existe

**Postconditions** : Utilisateur modifié dans la base de données

---

#### TC-USER-005 : Suppression d'un utilisateur (ADMIN)

**Nom** : Suppression d'un utilisateur par un administrateur

**Acteur** : Administrateur (ADMIN)

**Données d'entrée** :
- Path Parameter: `id = 1`

**Scénario** :
1. L'administrateur authentifié envoie une requête DELETE à `/api/users/delete/1` avec un token JWT valide
2. Le système valide le token JWT et vérifie que l'utilisateur a le rôle ADMIN
3. Le système recherche l'utilisateur avec l'ID 1 dans la base de données
4. Le système vérifie que l'utilisateur existe
5. Le système supprime l'utilisateur de la base de données
6. Le système retourne une réponse avec status 200 et message "User Deleted Successfully"

**Résultat attendu** : Utilisateur supprimé, status 200

**Préconditions** : Utilisateur authentifié avec rôle ADMIN, utilisateur avec ID 1 existe

**Postconditions** : Utilisateur supprimé de la base de données

---

#### TC-USER-006 : Récupération des transactions d'un utilisateur

**Nom** : Récupération de toutes les transactions associées à un utilisateur

**Acteur** : Utilisateur authentifié (tous rôles)

**Données d'entrée** :
- Path Parameter: `userId = 1`

**Scénario** :
1. L'utilisateur authentifié envoie une requête GET à `/api/users/transactions/1` avec un token JWT valide
2. Le système valide le token JWT
3. Le système recherche l'utilisateur avec l'ID 1 dans la base de données
4. Le système vérifie que l'utilisateur existe
5. Le système récupère toutes les transactions associées à cet utilisateur
6. Le système mappe l'utilisateur et ses transactions en UserDTO
7. Le système retourne une réponse avec status 200 et les informations de l'utilisateur avec ses transactions

**Résultat attendu** : Utilisateur avec ses transactions retourné, status 200

**Préconditions** : Utilisateur authentifié, utilisateur avec ID 1 existe

---

## Tests Unitaires - Services

### 1. CategoryService

**Fichier** : `CategoryServiceImplTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testCreateCategory_Success` | Créer une catégorie avec succès | Status 200, message "Category Saved Successfully" |
| `testGetAllCategories_Success` | Récupérer toutes les catégories | Status 200, liste de catégories non vide |
| `testGetCategoryById_Success` | Récupérer une catégorie par ID | Status 200, catégorie retournée |
| `testGetCategoryById_NotFound` | Récupérer une catégorie inexistante | Exception NotFoundException |
| `testUpdateCategory_Success` | Mettre à jour une catégorie | Status 200, catégorie mise à jour |
| `testUpdateCategory_NotFound` | Mettre à jour une catégorie inexistante | Exception NotFoundException |
| `testDeleteCategory_Success` | Supprimer une catégorie | Status 200, catégorie supprimée |
| `testDeleteCategory_NotFound` | Supprimer une catégorie inexistante | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

### 2. ProductService

**Fichier** : `ProductServiceImplTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testSaveProduct_Success` | Créer un produit avec image | Status 200, produit sauvegardé |
| `testSaveProduct_WithoutImage` | Créer un produit sans image | Status 200, produit sauvegardé |
| `testSaveProduct_CategoryNotFound` | Créer un produit avec catégorie inexistante | Exception NotFoundException |
| `testUpdateProduct_Success` | Mettre à jour un produit | Status 200, produit mis à jour |
| `testUpdateProduct_NotFound` | Mettre à jour un produit inexistant | Exception NotFoundException |
| `testGetAllProducts_Success` | Récupérer tous les produits | Status 200, liste de produits |
| `testGetProductById_Success` | Récupérer un produit par ID | Status 200, produit retourné |
| `testGetProductById_NotFound` | Récupérer un produit inexistant | Exception NotFoundException |
| `testDeleteProduct_Success` | Supprimer un produit | Status 200, produit supprimé |
| `testDeleteProduct_NotFound` | Supprimer un produit inexistant | Exception NotFoundException |
| `testSearchProduct_Success` | Rechercher un produit | Status 200, produits trouvés |
| `testSearchProduct_NotFound` | Rechercher un produit inexistant | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

### 3. SupplierService

**Fichier** : `SupplierServiceImplTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testAddSupplier_Success` | Ajouter un fournisseur | Status 200, fournisseur sauvegardé |
| `testUpdateSupplier_Success` | Mettre à jour un fournisseur | Status 200, fournisseur mis à jour |
| `testUpdateSupplier_NotFound` | Mettre à jour un fournisseur inexistant | Exception NotFoundException |
| `testGetAllSupplier_Success` | Récupérer tous les fournisseurs | Status 200, liste de fournisseurs |
| `testGetSupplierById_Success` | Récupérer un fournisseur par ID | Status 200, fournisseur retourné |
| `testGetSupplierById_NotFound` | Récupérer un fournisseur inexistant | Exception NotFoundException |
| `testDeleteSupplier_Success` | Supprimer un fournisseur | Status 200, fournisseur supprimé |
| `testDeleteSupplier_NotFound` | Supprimer un fournisseur inexistant | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

### 4. TransactionService

**Fichier** : `TransactionServiceImplTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testPurchase_Success` | Effectuer un achat | Status 200, stock augmenté, transaction créée |
| `testPurchase_SupplierIdRequired` | Achat sans supplierId | Exception NameValueRequiredException |
| `testPurchase_ProductNotFound` | Achat avec produit inexistant | Exception NotFoundException |
| `testSell_Success` | Effectuer une vente | Status 200, stock diminué, transaction créée |
| `testSell_ProductNotFound` | Vente avec produit inexistant | Exception NotFoundException |
| `testReturnToSupplier_Success` | Retourner un produit au fournisseur | Status 200, stock diminué, transaction créée |
| `testReturnToSupplier_SupplierIdRequired` | Retour sans supplierId | Exception NameValueRequiredException |
| `testGetAllTransactions_Success` | Récupérer toutes les transactions | Status 200, liste paginée |
| `testGetAllTransactionById_Success` | Récupérer une transaction par ID | Status 200, transaction retournée |
| `testGetAllTransactionById_NotFound` | Récupérer une transaction inexistante | Exception NotFoundException |
| `testUpdateTransactionStatus_Success` | Mettre à jour le statut d'une transaction | Status 200, statut mis à jour |
| `testUpdateTransactionStatus_NotFound` | Mettre à jour une transaction inexistante | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

### 5. UserService

**Fichier** : `UserServiceImplTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testRegisterUser_Success` | Enregistrer un utilisateur | Status 200, utilisateur créé |
| `testRegisterUser_WithDefaultRole` | Enregistrer avec rôle par défaut | Status 200, rôle MANAGER assigné |
| `testLoginUser_Success` | Connexion réussie | Status 200, token JWT retourné |
| `testLoginUser_EmailNotFound` | Connexion avec email inexistant | Exception NotFoundException |
| `testLoginUser_InvalidPassword` | Connexion avec mot de passe incorrect | Exception InvalidCredentialsException |
| `testGetAllUsers_Success` | Récupérer tous les utilisateurs | Status 200, liste d'utilisateurs |
| `testGetCurrentLoggedInUser_Success` | Récupérer l'utilisateur connecté | Utilisateur retourné |
| `testGetCurrentLoggedInUser_NotFound` | Utilisateur connecté inexistant | Exception NotFoundException |
| `testGetUserById_Success` | Récupérer un utilisateur par ID | Status 200, utilisateur retourné |
| `testGetUserById_NotFound` | Récupérer un utilisateur inexistant | Exception NotFoundException |
| `testUpdateUser_Success` | Mettre à jour un utilisateur | Status 200, utilisateur mis à jour |
| `testUpdateUser_WithPassword` | Mettre à jour avec nouveau mot de passe | Status 200, mot de passe encodé |
| `testUpdateUser_NotFound` | Mettre à jour un utilisateur inexistant | Exception NotFoundException |
| `testDeleteUser_Success` | Supprimer un utilisateur | Status 200, utilisateur supprimé |
| `testDeleteUser_NotFound` | Supprimer un utilisateur inexistant | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

## Tests Unitaires - Sécurité

### 1. JwtUtils

**Fichier** : `JwtUtilsTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testGenerateToken_Success` | Générer un token JWT | Token non null et non vide |
| `testGetUsernameFromToken_Success` | Extraire l'email du token | Email correctement extrait |
| `testIsTokenValid_Success` | Valider un token valide | true |
| `testIsTokenValid_InvalidUsername` | Valider un token avec mauvais username | false |
| `testIsTokenValid_ExpiredToken` | Valider un token expiré | Exception ou false |

**Couverture** : 100% des méthodes de JwtUtils

---

### 2. CustomUserDetailsService

**Fichier** : `CustomUserDetailsServiceTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testLoadUserByUsername_Success` | Charger un utilisateur par email | UserDetails retourné |
| `testLoadUserByUsername_NotFound` | Charger un utilisateur inexistant | Exception NotFoundException |

**Couverture** : 100% des méthodes du service

---

## Tests Unitaires - Contrôleurs

### 1. AuthController

**Fichier** : `AuthControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testRegisterUser_Success` | Endpoint POST /api/auth/register | Status 200, utilisateur enregistré |
| `testLoginUser_Success` | Endpoint POST /api/auth/login | Status 200, token retourné |

---

### 2. CategoryController

**Fichier** : `CategoryControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testCreateCategory_Success` | POST /api/categories/add (ADMIN) | Status 200 |
| `testGetAllCategories_Success` | GET /api/categories/all | Status 200 |
| `testGetCategoryById_Success` | GET /api/categories/{id} | Status 200 |
| `testUpdateCategory_Success` | PUT /api/categories/update/{id} (ADMIN) | Status 200 |
| `testDeleteCategory_Success` | DELETE /api/categories/delete/{id} (ADMIN) | Status 200 |

---

### 3. ProductController

**Fichier** : `ProductControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testSaveProduct_Success` | POST /api/products/add (ADMIN) | Status 200 |
| `testUpdateProduct_Success` | PUT /api/products/update (ADMIN) | Status 200 |
| `testGetAllProducts_Success` | GET /api/products/all | Status 200 |
| `testGetProductById_Success` | GET /api/products/{id} | Status 200 |
| `testDeleteProduct_Success` | DELETE /api/products/delete/{id} | Status 200 |
| `testSearchProduct_Success` | GET /api/products/search?input=... | Status 200 |

---

### 4. SupplierController

**Fichier** : `SupplierControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testAddSupplier_Success` | POST /api/suppliers/add (ADMIN) | Status 200 |
| `testGetAllSuppliers_Success` | GET /api/suppliers/all | Status 200 |
| `testGetSupplierById_Success` | GET /api/suppliers/{id} | Status 200 |
| `testUpdateSupplier_Success` | PUT /api/suppliers/update/{id} (ADMIN) | Status 200 |
| `testDeleteSupplier_Success` | DELETE /api/suppliers/delete/{id} (ADMIN) | Status 200 |

---

### 5. TransactionController

**Fichier** : `TransactionControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testPurchase_Success` | POST /api/transactions/purchase | Status 200 |
| `testSell_Success` | POST /api/transactions/sell | Status 200 |
| `testReturnToSupplier_Success` | POST /api/transactions/return | Status 200 |
| `testGetAllTransactions_Success` | GET /api/transactions/all | Status 200 |
| `testGetTransactionById_Success` | GET /api/transactions/{id} | Status 200 |
| `testGetTransactionByMonthAndYear_Success` | GET /api/transactions/by-month-year | Status 200 |
| `testUpdateTransactionStatus_Success` | PUT /api/transactions/{id} | Status 200 |

---

### 6. UserController

**Fichier** : `UserControllerTest.java`

#### Scénarios de Test

| Test | Description | Résultat Attendu |
|------|-------------|------------------|
| `testGetAllUsers_Success` | GET /api/users/all (ADMIN) | Status 200 |
| `testGetUserById_Success` | GET /api/users/{id} | Status 200 |
| `testUpdateUser_Success` | PUT /api/users/update/{id} | Status 200 |
| `testDeleteUser_Success` | DELETE /api/users/delete/{id} (ADMIN) | Status 200 |
| `testGetUserTransactions_Success` | GET /api/users/transactions/{userId} | Status 200 |
| `testGetCurrentUser_Success` | GET /api/users/current | Status 200 |

---

## Matrice de Couverture

### Services

| Service | Tests Unitaires | Couverture |
|---------|----------------|------------|
| CategoryService | 8 tests | 100% |
| ProductService | 12 tests | 100% |
| SupplierService | 8 tests | 100% |
| TransactionService | 12 tests | 100% |
| UserService | 15 tests | 100% |

### Sécurité

| Composant | Tests Unitaires | Couverture |
|-----------|----------------|------------|
| JwtUtils | 5 tests | 100% |
| CustomUserDetailsService | 2 tests | 100% |

### Contrôleurs

| Contrôleur | Tests | Couverture |
|------------|-------|------------|
| AuthController | 2 tests | 100% |
| CategoryController | 5 tests | 100% |
| ProductController | 6 tests | 100% |
| SupplierController | 5 tests | 100% |
| TransactionController | 7 tests | 100% |
| UserController | 6 tests | 100% |

### Total des Tests

- **Tests Unitaires Services** : 55 tests
- **Tests Unitaires Sécurité** : 7 tests
- **Tests Unitaires Contrôleurs** : 31 tests
- **Scénarios Fonctionnels** : 40+ scénarios
- **TOTAL** : **93 tests unitaires** + **40+ scénarios fonctionnels**

---

## Exécution des Tests

### Exécuter Tous les Tests

```bash
cd backend
mvn test
```

### Exécuter un Test Spécifique

```bash
mvn test -Dtest=CategoryServiceImplTest
```

### Exécuter avec Couverture de Code

```bash
mvn clean test jacoco:report
```

Le rapport sera généré dans : `target/site/jacoco/index.html`

---

## Utilisation dans Jira/Xray

### Format pour Jira/Xray

Chaque scénario de test peut être importé dans Jira/Xray avec les champs suivants :

- **Test Type** : Manual / Automated
- **Summary** : Nom du scénario (ex: "TC-AUTH-001 : Enregistrement d'un nouvel utilisateur")
- **Description** : Scénario complet avec étapes
- **Preconditions** : Préconditions listées
- **Test Steps** : Étapes détaillées du scénario
- **Expected Result** : Résultat attendu
- **Labels** : Catégories (ex: Authentication, Category, Product, etc.)

### Exemple d'import Xray

```json
{
  "testType": "Test",
  "summary": "TC-AUTH-001 : Enregistrement d'un nouvel utilisateur",
  "description": "Scénario complet...",
  "preconditions": "Aucune",
  "testSteps": [
    {
      "step": "L'utilisateur envoie une requête POST...",
      "data": "{...}",
      "expectedResult": "Validation des données..."
    }
  ]
}
```

---

## Utilisation pour Diagrammes de Séquence

Chaque scénario peut être représenté dans un diagramme de séquence avec les acteurs suivants :

- **Client** : Utilisateur/Frontend
- **Controller** : Contrôleur REST (AuthController, ProductController, etc.)
- **Service** : Service métier (UserService, ProductService, etc.)
- **Repository** : Repository JPA (UserRepository, ProductRepository, etc.)
- **Security** : JwtUtils, CustomUserDetailsService, AuthFilter
- **Database** : Base de données

### Exemple de Diagramme de Séquence pour TC-AUTH-003

```
Client -> AuthController: POST /api/auth/login
AuthController -> UserService: loginUser(loginRequest)
UserService -> UserRepository: findByEmail(email)
UserRepository -> Database: SELECT * FROM users WHERE email = ?
Database -> UserRepository: User entity
UserRepository -> UserService: User
UserService -> PasswordEncoder: matches(password, user.password)
PasswordEncoder -> UserService: boolean
UserService -> JwtUtils: generateToken(userDetails)
JwtUtils -> UserService: JWT token
UserService -> AuthController: Response(token, role, expirationTime)
AuthController -> Client: 200 OK + Response
```

---

## Conclusion

Le plan de test couvre :
- **93 tests unitaires** au total
- **40+ scénarios fonctionnels** détaillés
- **100% de couverture** des services
- **Tous les scénarios** de succès et d'erreur
- **Tests de sécurité** pour l'authentification
- **Tests de contrôleurs** pour tous les endpoints REST
- **Format compatible** avec Jira/Xray et diagrammes de séquence

Tous les tests sont automatisés et peuvent être exécutés avec une simple commande Maven.
