# Centralcee
## Travail d'algorithmes final - Gestion de Réseau et télécommunications RIT

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_SALLES 5
#define MAX_APPAREILS 50

typedef struct {
    int id;
    char nom[50];
    char ip[20];
    char type[20];   // PC, serveur, routeur
    int statut;      // 1 = connecté, 0 = déconnecté
    int trafic;      // en Mo
} Appareil;

typedef struct {
    int idSalle;
    char nomSalle[50];
    Appareil appareils[MAX_APPAREILS];
    int nbAppareils;
    int traficTotal;
} Salle;

Salle reseau[MAX_SALLES];

// ------------------ Fonctions ------------------

void initialiserReseau() {
    for (int i = 0; i < MAX_SALLES; i++) {
        reseau[i].idSalle = i + 1;
        sprintf(reseau[i].nomSalle, "Salle %d", i + 1);
        reseau[i].nbAppareils = 0;
        reseau[i].traficTotal = 0;
    }
}

void ajouterAppareil() {
    int salleChoisie;
    printf("Choisir une salle (1-5): ");
    scanf("%d", &salleChoisie);
    if (salleChoisie < 1 || salleChoisie > MAX_SALLES) {
        printf("Salle invalide.\n");
        return;
    }
    Salle *salle = &reseau[salleChoisie - 1];
    if (salle->nbAppareils >= MAX_APPAREILS) {
        printf("Salle pleine.\n");
        return;
    }
    Appareil a;
    printf("ID: "); scanf("%d", &a.id);
    printf("Nom: "); scanf("%s", a.nom);
    printf("IP: "); scanf("%s", a.ip);
    printf("Type: "); scanf("%s", a.type);
    printf("Statut (1 = connecté, 0 = déconnecté): "); scanf("%d", &a.statut);
    a.trafic = 0;
    salle->appareils[salle->nbAppareils++] = a;
    printf("Appareil ajouté avec succès.\n");
}

void afficherAppareils() {
    int salleChoisie;
    printf("Choisir une salle (1-5): ");
    scanf("%d", &salleChoisie);
    if (salleChoisie < 1 || salleChoisie > MAX_SALLES) return;
    
    Salle *salle = &reseau[salleChoisie - 1];
    if (salle->nbAppareils == 0) {
        printf("Aucun appareil dans cette salle.\n");
        return;
    }
    for (int i = 0; i < salle->nbAppareils; i++) {
        Appareil a = salle->appareils[i];
        printf("ID:%d | Nom:%s | IP:%s | Type:%s | Statut:%s | Trafic:%d Mo\n",
               a.id, a.nom, a.ip, a.type,
               a.statut ? "Connecté" : "Déconnecté",
               a.trafic);
    }
}

void rechercherAppareil() {
    char ip[20];
    printf("Entrer IP: ");
    scanf("%s", ip);
    for (int i = 0; i < MAX_SALLES; i++) {
        for (int j = 0; j < reseau[i].nbAppareils; j++) {
            if (strcmp(reseau[i].appareils[j].ip, ip) == 0) {
                Appareil a = reseau[i].appareils[j];
                printf("Trouvé: ID:%d Nom:%s Salle:%s Statut:%s Trafic:%d Mo\n",
                       a.id, a.nom, reseau[i].nomSalle,
                       a.statut ? "Connecté" : "Déconnecté",
                       a.trafic);
                return;
            }
        }
    }
    printf("Appareil non trouvé.\n");
}

void changerStatut() {
    char ip[20];
    printf("Entrer IP: ");
    scanf("%s", ip);
    for (int i = 0; i < MAX_SALLES; i++) {
        for (int j = 0; j < reseau[i].nbAppareils; j++) {
            if (strcmp(reseau[i].appareils[j].ip, ip) == 0) {
                reseau[i].appareils[j].statut = !reseau[i].appareils[j].statut;
                printf("Statut modifié avec succès.\n");
                return;
            }
        }
    }
    printf("Appareil non trouvé.\n");
}

void simulerTrafic() {
    srand(time(NULL));
    for (int i = 0; i < MAX_SALLES; i++) {
        reseau[i].traficTotal = 0;
        for (int j = 0; j < reseau[i].nbAppareils; j++) {
            if (reseau[i].appareils[j].statut == 1) {
                int ajout = 10 + rand() % 41; // 10-50 Mo
                reseau[i].appareils[j].trafic += ajout;
            }
            reseau[i].traficTotal += reseau[i].appareils[j].trafic;
        }
        if (reseau[i].traficTotal > 1000) {
            printf("ALERTE: %s dépasse 1000 Mo!\n", reseau[i].nomSalle);
        }
    }
    printf("Simulation terminée.\n");
}

void afficherStatistiques() {
    int totalAppareils = 0;
    int totalTrafic = 0;
    for (int i = 0; i < MAX_SALLES; i++) {
        Salle *salle = &reseau[i];
        printf("\n--- %s ---\n", salle->nomSalle);
        printf("Nombre d'appareils: %d\n", salle->nbAppareils);
        int connectes = 0;
        int maxTrafic = 0;
        char appareilMax[50] = "";
        for (int j = 0; j < salle->nbAppareils; j++) {
            if (salle->appareils[j].statut == 1) connectes++;
            if (salle->appareils[j].trafic > maxTrafic) {
                maxTrafic = salle->appareils[j].trafic;
                strcpy(appareilMax, salle->appareils[j].nom);
            }
        }
        printf("Appareils connectés: %d\n", connectes);
        printf("Trafic total: %d Mo\n", salle->traficTotal);
        if (salle->nbAppareils > 0)
            printf("Plus gros trafic: %s (%d Mo)\n", appareilMax, maxTrafic);
        totalAppareils += salle->nbAppareils;
        totalTrafic += salle->traficTotal;
    }
    printf("\n=== Statistiques Réseau Global ===\n");
    printf("Total appareils: %d | Total Trafic: %d Mo\n", totalAppareils, totalTrafic);
}

void trierAppareils() {
    int salleChoisie, critere;
    printf("Choisir une salle (1-5): ");
    scanf("%d", &salleChoisie);
    if (salleChoisie < 1 || salleChoisie > MAX_SALLES) return;
    
    Salle *salle = &reseau[salleChoisie - 1];
    if (salle->nbAppareils == 0) return;

    printf("Critère de tri (1=ID, 2=Trafic): ");
    scanf("%d", &critere);
    for (int i = 0; i < salle->nbAppareils - 1; i++) {
        for (int j = i + 1; j < salle->nbAppareils; j++) {
            int condition = (critere == 1) ? (salle->appareils[i].id > salle->appareils[j].id) : (salle->appareils[i].trafic < salle->appareils[j].trafic);
            if (condition) {
                Appareil temp = salle->appareils[i];
                salle->appareils[i] = salle->appareils[j];
                salle->appareils[j] = temp;
            }
        }
    }
    printf("Appareils triés avec succès.\n");
}

// ------------------ Menu Principal ------------------

int main() {
    initialiserReseau();
    int choix;
    do {
        printf("\n====== MENU RESEAU ======\n");
        printf("1. Ajouter appareil | 2. Afficher salle | 3. Rechercher (IP)\n");
        printf("4. Changer statut  | 5. Simuler trafic | 6. Statistiques\n");
        printf("7. Trier           | 8. Quitter\n");
        printf("Choix: ");
        scanf("%d", &choix);
        switch (choix) {
            case 1: ajouterAppareil(); break;
            case 2: afficherAppareils(); break;
            case 3: rechercherAppareil(); break;
            case 4: changerStatut(); break;
            case 5: simulerTrafic(); break;
            case 6: afficherStatistiques(); break;
            case 7: trierAppareils(); break;
            case 8: printf("Au revoir!\n"); break;
        }
    } while (choix != 8);
    return 0;
}
