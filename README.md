# Plan de Test - Inventory Management System

Ce document contient tous les scénarios, structurés pour créer des diagrammes de séquence en boîte noire et boîte blanche.

---

## Vue d'ensemble

### Types de Tests

- **Scénarios Fonctionnels Complets** : Scénarios end-to-end pour chaque entité (CRUD complet)
- **Tests Unitaires** : Tests isolés des services, contrôleurs et composants de sécurité
- **Tests de Contrôleurs** : Tests des endpoints REST avec MockMvc

### Diagrammes de Séquence

- **Boîte Noire** : Vue externe (Client ↔ API)
- **Boîte Blanche** : Vue interne (Client → Controller → Service → Repository → Database)

### Outils Utilisés

- **JUnit 5** : Framework de test
- **Mockito** : Framework de mocking
- **Spring Boot Test** : Support de test Spring
- **H2 Database** : Base de données en mémoire pour les tests
- **Jira/Xray** : Gestion des tests et traçabilité

---

## Scénarios de Cas d'Utilisation

### 1-<< Gestion des Catégories >> :

**A-Scénario pour << Gestion des Catégories >> :**

**Nom** : Gestion des catégories

**Acteur** : Admin

**Donnée d'entrée** : Le cas d'utilisation commence lorsque l'administrateur clique sur le bouton << Catégories >> dans le menu de navigation.

**Scénario principale** :

1. Le système affiche la liste des catégories avec la possibilité d'ajouter, modifier, supprimer ou consulter une catégorie.

2. L'administrateur choisit d'ajouter une catégorie.

3. Le système demande à l'administrateur de saisir le nom de la catégorie.

4. L'administrateur remplit ces informations et clique sur ajouter.

5. Le système vérifie si tous les champs sont remplis et valides.

6. Le système sauvegarde la catégorie dans la base de données.

7. Le système informe l'administrateur que l'opération a été effectuée avec succès.

**Scénario erreur** :

**Champs vide** :

6a. Le système informe l'administrateur qu'il doit remplir tous les champs avant d'ajouter. Retour à l'étape 3.

**Catégorie déjà existante** :

6b. Le système informe l'administrateur que cette catégorie existe déjà. Retour à l'étape 3.

**Scénario alternatif** :

**Modifier catégorie** :

2a. L'administrateur choisit de modifier une catégorie existante.

3a. Le système affiche le formulaire pré-rempli avec les informations de la catégorie sélectionnée.

4a. L'administrateur modifie les informations et clique sur modifier. Le scénario reprend à l'étape 5.

**Supprimer catégorie** :

2b. L'administrateur choisit de supprimer une catégorie.

3b. Le système demande confirmation à l'administrateur.

4b. L'administrateur confirme la suppression.

5b. Le système vérifie si la catégorie est utilisée par des produits.

6b. Si la catégorie n'est pas utilisée, le système supprime la catégorie de la base de données.

7b. Le système informe l'administrateur que la suppression a été effectuée avec succès. Le scénario reprend à l'étape 1.

**Consulter catégorie** :

2c. L'administrateur choisit de consulter les détails d'une catégorie.

3c. Le système affiche les détails de la catégorie (nom, nombre de produits associés). Le scénario reprend à l'étape 1.

---

### 2-<< Gestion des Produits >> :

**A-Scénario pour << Gestion des Produits >> :**

**Nom** : Gestion des produits

**Acteur** : Admin

**Donnée d'entrée** : Le cas d'utilisation commence lorsque l'administrateur clique sur le bouton << Produits >> dans le menu de navigation.

**Scénario principale** :

1. Le système affiche la liste des produits avec la possibilité d'ajouter, modifier, supprimer, rechercher ou consulter un produit.

2. L'administrateur choisit d'ajouter un produit.

3. Le système demande à l'administrateur de saisir le nom, le SKU, le prix, la quantité en stock, la description, de sélectionner une catégorie et d'uploader une image (optionnel).

4. L'administrateur remplit ces informations et clique sur ajouter.

5. Le système vérifie si tous les champs obligatoires sont remplis et valides.

6. Le système vérifie que la catégorie sélectionnée existe dans la base de données.

7. Si une image est fournie, le système sauvegarde l'image dans le répertoire `/public/products/` avec un nom unique.

8. Le système sauvegarde le produit dans la base de données.

9. Le système informe l'administrateur que l'opération a été effectuée avec succès.

**Scénario erreur** :

**Champs obligatoires manquants** :

5a. Le système informe l'administrateur qu'il doit remplir tous les champs obligatoires (nom, SKU, prix, quantité, catégorie) avant d'ajouter. Retour à l'étape 3.

**Catégorie inexistante** :

6a. Le système informe l'administrateur que la catégorie sélectionnée n'existe pas. Retour à l'étape 3.

**SKU déjà existant** :

5b. Le système vérifie si le SKU existe déjà dans la base de données.

5c. Si le SKU existe, le système informe l'administrateur que ce SKU est déjà utilisé. Retour à l'étape 3.

**Erreur lors du téléchargement de l'image** :

7a. Le système informe l'administrateur qu'une erreur s'est produite lors du téléchargement de l'image. Retour à l'étape 3.

**Scénario alternatif** :

**Modifier produit** :

2a. L'administrateur choisit de modifier un produit existant.

3a. Le système affiche le formulaire pré-rempli avec les informations du produit sélectionné.

4a. L'administrateur modifie les informations (peut changer l'image) et clique sur modifier. Le scénario reprend à l'étape 5.

**Supprimer produit** :

2b. L'administrateur choisit de supprimer un produit.

3b. Le système demande confirmation à l'administrateur.

4b. L'administrateur confirme la suppression.

5b. Le système supprime l'image associée du répertoire `/public/products/` si elle existe.

6b. Le système supprime le produit de la base de données.

7b. Le système informe l'administrateur que la suppression a été effectuée avec succès. Le scénario reprend à l'étape 1.

**Rechercher produit** :

2c. L'administrateur choisit de rechercher un produit.

3c. Le système demande à l'administrateur de saisir un terme de recherche (nom ou description).

4c. L'administrateur saisit le terme de recherche et clique sur rechercher.

5c. Le système affiche la liste des produits correspondant au terme de recherche. Le scénario reprend à l'étape 1.

**Consulter produit** :

2d. L'administrateur choisit de consulter les détails d'un produit.

3d. Le système affiche les détails du produit (nom, SKU, prix, stock, catégorie, description, image). Le scénario reprend à l'étape 1.

---

### 3-<< Gestion des Fournisseurs >> :

**A-Scénario pour << Gestion des Fournisseurs >> :**

**Nom** : Gestion des fournisseurs

**Acteur** : Admin

**Donnée d'entrée** : Le cas d'utilisation commence lorsque l'administrateur clique sur le bouton << Fournisseurs >> dans le menu de navigation.

**Scénario principale** :

1. Le système affiche la liste des fournisseurs avec la possibilité d'ajouter, modifier, supprimer ou consulter un fournisseur.

2. L'administrateur choisit d'ajouter un fournisseur.

3. Le système demande à l'administrateur de saisir le nom, les informations de contact et l'adresse du fournisseur.

4. L'administrateur remplit ces informations et clique sur ajouter.

5. Le système vérifie si tous les champs obligatoires sont remplis (nom et informations de contact).

6. Le système sauvegarde le fournisseur dans la base de données.

7. Le système informe l'administrateur que l'opération a été effectuée avec succès.

**Scénario erreur** :

**Champs obligatoires manquants** :

5a. Le système informe l'administrateur qu'il doit remplir tous les champs obligatoires (nom et informations de contact) avant d'ajouter. Retour à l'étape 3.

**Fournisseur déjà existant** :

5b. Le système vérifie si un fournisseur avec le même nom existe déjà.

5c. Si le fournisseur existe, le système informe l'administrateur que ce fournisseur existe déjà. Retour à l'étape 3.

**Scénario alternatif** :

**Modifier fournisseur** :

2a. L'administrateur choisit de modifier un fournisseur existant.

3a. Le système affiche le formulaire pré-rempli avec les informations du fournisseur sélectionné.

4a. L'administrateur modifie les informations et clique sur modifier. Le scénario reprend à l'étape 5.

**Supprimer fournisseur** :

2b. L'administrateur choisit de supprimer un fournisseur.

3b. Le système demande confirmation à l'administrateur.

4b. L'administrateur confirme la suppression.

5b. Le système vérifie si le fournisseur est utilisé dans des transactions.

6b. Si le fournisseur n'est pas utilisé, le système supprime le fournisseur de la base de données.

7b. Le système informe l'administrateur que la suppression a été effectuée avec succès. Le scénario reprend à l'étape 1.

**Consulter fournisseur** :

2c. L'administrateur choisit de consulter les détails d'un fournisseur.

3c. Le système affiche les détails du fournisseur (nom, informations de contact, adresse, nombre de transactions associées). Le scénario reprend à l'étape 1.

---

### 4-<< Gestion des Transactions >> :

**A-Scénario pour << Gestion des Transactions >> :**

**Nom** : Gestion des transactions

**Acteur** : Utilisateur authentifié (Manager, Admin)

**Donnée d'entrée** : Le cas d'utilisation commence lorsque l'utilisateur authentifié clique sur le bouton << Transactions >> dans le menu de navigation.

**Scénario principale (Achat - Purchase)** :

1. Le système affiche la liste des transactions avec la possibilité d'effectuer un achat, une vente, un retour au fournisseur, consulter une transaction ou filtrer les transactions.

2. L'utilisateur choisit d'effectuer un achat (Purchase).

3. Le système demande à l'utilisateur de sélectionner un produit, saisir la quantité, sélectionner un fournisseur, et optionnellement saisir une description et une note.

4. L'utilisateur remplit ces informations et clique sur confirmer l'achat.

5. Le système vérifie si tous les champs obligatoires sont remplis (produit, quantité, fournisseur).

6. Le système vérifie que le produit et le fournisseur existent dans la base de données.

7. Le système récupère l'utilisateur connecté depuis le contexte de sécurité.

8. Le système crée une transaction de type PURCHASE et augmente le stock du produit de la quantité achetée.

9. Le système sauvegarde la transaction dans la base de données.

10. Le système met à jour le stock du produit dans la base de données.

11. Le système informe l'utilisateur que l'opération a été effectuée avec succès.

**Scénario erreur** :

**Champs obligatoires manquants** :

5a. Le système informe l'utilisateur qu'il doit remplir tous les champs obligatoires (produit, quantité, fournisseur) avant de confirmer. Retour à l'étape 3.

**Produit inexistant** :

6a. Le système informe l'utilisateur que le produit sélectionné n'existe pas. Retour à l'étape 3.

**Fournisseur inexistant** :

6b. Le système informe l'utilisateur que le fournisseur sélectionné n'existe pas. Retour à l'étape 3.

**Quantité invalide** :

5b. Le système vérifie si la quantité est positive.

5c. Si la quantité est négative ou nulle, le système informe l'utilisateur que la quantité doit être positive. Retour à l'étape 3.

**Scénario alternatif** :

**Effectuer une vente (Sell)** :

2a. L'utilisateur choisit d'effectuer une vente.

3a. Le système demande à l'utilisateur de sélectionner un produit, saisir la quantité, et optionnellement saisir une description et une note.

4a. L'utilisateur remplit ces informations et clique sur confirmer la vente.

5a. Le système vérifie si tous les champs obligatoires sont remplis (produit, quantité).

6a. Le système vérifie que le produit existe et que le stock disponible est suffisant.

7a. Le système récupère l'utilisateur connecté.

8a. Le système crée une transaction de type SELL et diminue le stock du produit de la quantité vendue.

9a. Le système sauvegarde la transaction et met à jour le stock. Le scénario reprend à l'étape 11.

**Stock insuffisant** :

6aa. Le système informe l'utilisateur que le stock disponible est insuffisant pour cette vente. Retour à l'étape 3a.

**Retour au fournisseur (Return)** :

2b. L'utilisateur choisit de retourner un produit au fournisseur.

3b. Le système demande à l'utilisateur de sélectionner un produit, saisir la quantité, sélectionner un fournisseur, et optionnellement saisir une description et une note.

4b. L'utilisateur remplit ces informations et clique sur confirmer le retour.

5b. Le système vérifie si tous les champs obligatoires sont remplis (produit, quantité, fournisseur).

6b. Le système vérifie que le produit et le fournisseur existent et que le stock disponible est suffisant.

7b. Le système récupère l'utilisateur connecté.

8b. Le système crée une transaction de type RETURN et diminue le stock du produit de la quantité retournée.

9b. Le système sauvegarde la transaction et met à jour le stock. Le scénario reprend à l'étape 11.

**Consulter transaction** :

2c. L'utilisateur choisit de consulter les détails d'une transaction.

3c. Le système affiche les détails de la transaction (type, produit, quantité, fournisseur si applicable, utilisateur, statut, date, description, note). Le scénario reprend à l'étape 1.

**Modifier statut d'une transaction** :

2d. L'utilisateur choisit de modifier le statut d'une transaction.

3d. Le système affiche la liste des statuts disponibles (PENDING, COMPLETED, CANCELLED).

4d. L'utilisateur sélectionne un nouveau statut et clique sur modifier.

5d. Le système met à jour le statut de la transaction dans la base de données.

6d. Le système informe l'utilisateur que le statut a été modifié avec succès. Le scénario reprend à l'étape 1.

**Filtrer transactions** :

2e. L'utilisateur choisit de filtrer les transactions.

3e. Le système demande à l'utilisateur de sélectionner un filtre (par type, par mois/année, par statut).

4e. L'utilisateur sélectionne un filtre et saisit les critères si nécessaire.

5e. Le système affiche la liste des transactions filtrées. Le scénario reprend à l'étape 1.

---

### 5-<< Authentification et Gestion Utilisateurs >> :

**A-Scénario pour << Authentification et Gestion Utilisateurs >> :**

**Nom** : Authentification et gestion des utilisateurs

**Acteur** : Utilisateur non authentifié, Utilisateur authentifié, Admin

**Donnée d'entrée** : Le cas d'utilisation commence lorsque l'utilisateur accède à l'application ou clique sur le bouton << Utilisateurs >> dans le menu de navigation (pour les admins).

**Scénario principale (Enregistrement)** :

1. L'utilisateur non authentifié accède à la page d'enregistrement.

2. Le système affiche le formulaire d'enregistrement avec les champs : nom, email, mot de passe, numéro de téléphone, et optionnellement le rôle.

3. L'utilisateur remplit ces informations et clique sur s'enregistrer.

4. Le système vérifie si tous les champs obligatoires sont remplis (nom, email, mot de passe, numéro de téléphone).

5. Le système vérifie le format de l'email.

6. Le système vérifie si l'email existe déjà dans la base de données.

7. Le système encode le mot de passe avec BCrypt.

8. Le système crée un nouvel utilisateur avec le rôle spécifié (ou MANAGER par défaut si aucun rôle n'est fourni).

9. Le système sauvegarde l'utilisateur dans la base de données.

10. Le système informe l'utilisateur que l'enregistrement a été effectué avec succès.

**Scénario erreur** :

**Champs obligatoires manquants** :

4a. Le système informe l'utilisateur qu'il doit remplir tous les champs obligatoires avant de s'enregistrer. Retour à l'étape 2.

**Format email invalide** :

5a. Le système informe l'utilisateur que le format de l'email est invalide. Retour à l'étape 2.

**Email déjà existant** :

6a. Le système informe l'utilisateur que cet email est déjà utilisé. Retour à l'étape 2.

**Scénario alternatif** :

**Connexion (Login)** :

1a. L'utilisateur accède à la page de connexion.

2a. Le système affiche le formulaire de connexion avec les champs : email et mot de passe.

3a. L'utilisateur saisit son email et son mot de passe et clique sur se connecter.

4a. Le système vérifie si tous les champs sont remplis.

5a. Le système recherche l'utilisateur par email dans la base de données.

6a. Le système vérifie que l'utilisateur existe.

7a. Le système compare le mot de passe fourni avec le hash stocké.

8a. Si le mot de passe correspond, le système génère un token JWT avec les informations de l'utilisateur.

9a. Le système retourne le token JWT, le rôle de l'utilisateur et le temps d'expiration.

10a. Le système redirige l'utilisateur vers la page d'accueil. Le scénario se termine.

**Email inexistant** :

5aa. Le système informe l'utilisateur que l'email n'existe pas. Retour à l'étape 2a.

**Mot de passe incorrect** :

7aa. Le système informe l'utilisateur que le mot de passe est incorrect. Retour à l'étape 2a.

**Consulter profil utilisateur** :

1b. L'utilisateur authentifié clique sur son profil.

2b. Le système récupère l'utilisateur connecté depuis le contexte de sécurité.

3b. Le système affiche les informations de l'utilisateur (nom, email, numéro de téléphone, rôle, date de création). Le scénario se termine.

**Modifier profil utilisateur** :

1c. L'utilisateur authentifié choisit de modifier son profil.

2c. Le système affiche le formulaire pré-rempli avec les informations de l'utilisateur connecté.

3c. L'utilisateur modifie les informations (nom, numéro de téléphone, et optionnellement le mot de passe) et clique sur modifier.

4c. Le système vérifie si tous les champs sont valides.

5c. Si un nouveau mot de passe est fourni, le système l'encode avec BCrypt.

6c. Le système met à jour l'utilisateur dans la base de données.

7c. Le système informe l'utilisateur que la modification a été effectuée avec succès. Le scénario se termine.

**Consulter tous les utilisateurs (Admin uniquement)** :

1d. L'administrateur clique sur le bouton << Utilisateurs >> dans le menu.

2d. Le système vérifie que l'utilisateur a le rôle ADMIN.

3d. Le système affiche la liste de tous les utilisateurs avec la possibilité de consulter, modifier ou supprimer un utilisateur.

4d. L'administrateur peut consulter les détails d'un utilisateur, modifier un utilisateur ou supprimer un utilisateur. Le scénario se termine.

**Supprimer utilisateur (Admin uniquement)** :

1e. L'administrateur choisit de supprimer un utilisateur depuis la liste.

2e. Le système demande confirmation à l'administrateur.

3e. L'administrateur confirme la suppression.

4e. Le système vérifie que l'utilisateur existe.

5e. Le système supprime l'utilisateur de la base de données.

6e. Le système informe l'administrateur que la suppression a été effectuée avec succès. Le scénario se termine.

---
