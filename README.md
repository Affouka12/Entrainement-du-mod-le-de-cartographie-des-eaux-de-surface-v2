# Entrainement-du-mod-le-de-cartographie-des-eaux-de-surface-v2


# -*- coding: utf-8 -*-
"""
Cartographie des eaux de surface en CÔTE D'IVOIRE - Sentinel-1 + U-Net (encodeur ResNet50 pré-entraîné)
Version 2 : adaptée au contexte ivoirien

Adaptations Côte d'Ivoire
  - 8 zones d'étude : lagunes (Ébrié, Aby, Grand-Lahou), lacs de barrage (Ayamé, Taabo, Kossou, Buyo)
    et savane nord (Korhogo : retenues, bas-fonds, riziculture)
  - Fuseau UTM choisi automatiquement par zone (29N à l'ouest de -6°, 30N à l'est) -> EPSG:32629 / 32630
  - Dates d'entraînement réparties sur les saisons ivoiriennes (grande/petite saison sèche et des pluies)
  - Étiquettes S2 : MNDWI + NIR bas + pente < 15° (limite les ombres de relief / forêt)
  - Zones de TEST totalement exclues de l'entraînement (Grand-Lahou, Lac de Buyo) => F1 de généralisation honnête
  - Suivi temporel : journal CSV des surfaces en eau à chaque exécution quasi temps réel

Utilisation : copier chaque bloc "# %%" dans une cellule Google Colab (GPU T4 : Exécution > Modifier le type).
Le F1 = 0.92 est un OBJECTIF : il se mesure (cellule 8), il ne se garantit pas.
"""

# mask = near_real_time("Lac_Kossou")
# for z in ["Ebrie_Abidjan", "Lac_Kossou", "Lac_Buyo", "Korhogo_nord"]: near_real_time(z)
 
# %% [9] Notes
"""
- Lisez surtout les lignes "TEST Grand_Lahou / Lac_Buyo" : ce sont des zones jamais vues, donc le vrai F1.
  Un F1 val de 0.95 avec un F1 test de 0.80 = surapprentissage au site, pas un bon modèle.
- Cas difficiles en Côte d'Ivoire : mangroves et végétation inondée (rétrodiffusion forte, "double bounce"),
  jacinthe d'eau en lagune, rizières inondées au nord, eau turbide ou à vent fort (surface rugueuse = moins sombre en SAR).
- Pour viser 0.92 : numériser ou corriger à la main 50-100 tuiles (surtout lagunes, bas-fonds, rizières), puis
  fine-tuner dessus. Les étiquettes MNDWI sont "faibles" et limitent le F1 mesurable.
- Plus de zones = meilleure généralisation (Comoé, Cavally, Sassandra, lagune de Grand-Bassam, Lac de Kossou nord).
- Les emprises des AOIS sont approximatives : affichez-les avec geemap avant de lancer l'extraction.
- Temps réel : Colab n'est pas un service permanent. Pour automatiser, utilisez le modèle sauvegardé dans un job
  planifié (Cloud Run / GitHub Actions) déclenché toutes les ~6 jours.
"""
