# Git werkwijze

**Volg deze werkwijze wanneer je lokaal wijzigingen wilt maken en pushen naar de master-branch**



1. Alvorens je begint te werken zorg je lokaal voor de laatste versie van de repository. De laatste updates kan u binnenhalen met:
```
$ git pull
```

2. Je werkt lokaal aan nieuwe code en wenst deze toe te voegen aan de master-branch
```bash
$ git add .
# Je kan ook 1 of meerder bestanden toevoegen door de naam ervan mee te geven in plaats van het .
```

3. Geef een commit-boodschap mee aan de wijzigingen die je hebt gemaakt:
```bash
$ git commit -m"commit message"
```

4. Om conflicten te vermijden haal je nog eens de repository binnen alvorens je jouw wijzigingen pusht, dit doen we als volgt:
```
$ git pull --rebase
```

5. Je pusht je wijzigingen naar de repository
```
$ git push
```


&copy; Lennert Mertens  
&copy; Rob Eggermont  
