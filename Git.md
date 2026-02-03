Je vais t'expliquer cette séquence d'opérations Git qui permet de synchroniser plusieurs branches avec une base commune avant de les intégrer.

## Créer une branche staging à partir de develop

Cette opération crée une nouvelle branche `staging` qui pointe initialement sur le même commit que `develop` . Tu exécutes :

```bash
git checkout develop
git pull origin develop
git checkout -b staging
```

La branche `staging` sert d'environnement d'intégration où tu vas fusionner plusieurs branches avant de les déployer ou de les merger dans `develop` .

## Rebase de staging à partir de la branche symfony

Le rebase de `staging` sur `symfony` réapplique tous les commits de `staging` (qui viennent de `develop`) au-dessus des commits de la branche `symfony` . Cela change la base historique de `staging` :

```bash
git checkout staging
git rebase symfony
```

Après cette opération, `staging` contient d'abord tous les commits de `symfony`, puis les commits qui étaient spécifiques à `develop` . Si des conflits apparaissent, tu dois les résoudre avec `git add` puis `git rebase --continue` .

## Rebase des 4 branches de tickets à partir de symfony

Cette étape synchronise chaque branche de ticket avec la branche `symfony` pour qu'elles partagent la même base . Pour chaque branche :

```bash
git checkout branche-ticket-1
git rebase symfony

git checkout branche-ticket-2
git rebase symfony

git checkout branche-ticket-3
git rebase symfony

git checkout branche-ticket-4
git rebase symfony
```

Chaque branche de ticket aura maintenant l'historique complet de `symfony` suivi de ses propres commits . Cela évite les conflits complexes lors du merge final et assure que tous les tickets sont compatibles avec la nouvelle base `symfony` .

## Merge des 4 branches sur staging

Finalement, tu intègres les modifications de chaque branche de ticket dans `staging` avec des merges classiques  :

```bash
git checkout staging
git merge branche-ticket-1
git merge branche-ticket-2
git merge branche-ticket-3
git merge branche-ticket-4
```

Le merge crée des commits de fusion qui préservent l'historique de chaque branche . Puisque toutes les branches (staging et les 4 tickets) ont été rebasées sur `symfony`, les merges devraient être fluides avec peu ou pas de conflits .

**Avantage de cette approche** : Elle garantit que `staging` contient la base `symfony` avec tous les développements des 4 tickets intégrés de manière cohérente, tout en maintenant une traçabilité claire de chaque ticket via les commits de merge .

Sources
