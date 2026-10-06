# Pr-ctica-LogAnalyzer
Sistema de Processament de Logs Distribuït (LogAnalyzer)

1. Context del Problema
Una empresa de ciberseguretat rep diàriament grans volums de fitxers de registre (logs) des de múltiples servidors. Per evitar saturar la memòria RAM de l'aplicació principal i aprofitar la potència del sistema operatiu, es requereix dissenyar un sistema multiprocés en Java: un programa central (AnalitzadorPrincipal) que actua com a orquestrador (pare) i delega el filtrat i comptatge a processos independents (FiltreLog) mitjançant redirecció de fluxos i canonades (IPC).
2. Disseny Gradual per Nivells (Escala Incremental)
Nivell 1: Comunicació Bàsica Pare-Fill
FiltreLog.java (Procés Fill):
Rep un text línia per línia a través de la seva entrada estàndard (System.in).
Per defecte, compta quantes vegades apareix la paraula clau ERROR.
Envia el resultat numèric final com una única línia a la seva sortida estàndard (System.out).
AnalitzadorPrincipal.java (Procés Pare):
Instancia el procés fill executant la classe FiltreLog mitjançant ProcessBuilder.
Envia un text de prova de múltiples línies al fill a través del seu OutputStream.
Executa out.flush() i out.close() per indicar la fi de les dades.
Llegeix la resposta del fill pel seu InputStream, la mostra per pantalla i espera la finalització amb p.waitFor().
Nivell 2: Gestió d'Errors i Control d'Execució
Validació i Codi de Sortida en el Fill:
Si FiltreLog rep un text buit o nul per System.in, ha d'emetre un missatge d'error per System.err ("Error: Text buit") i finalitzar immediatament amb System.exit(1). Si tot és correcte, finalitza amb System.exit(0).
Configuració del Pare:
Redirigeix la sortida d'errors del fill (stderr) cap a un fitxer físic anomenat errors_filtre.log utilitzant pb.redirectError(File).
Captura el codi de finalització del fill amb p.exitValue().
Imprimeix per pantalla l'estat final de l'execució segons el codi retornat.
Nivell 3: Pipeline Multiprocés en Paral·lel
L'aplicació principal ha d'orquestrar 2 processos fills executats simultàniament:
Fill A (FiltreLog ERROR): Filtra i compta les línies que contenen la paraula ERROR.
Fill B (FiltreLog WARNING): Filtra i compta les línies que contenen la paraula WARNING.
Requisit d'Arguments: Modifica FiltreLog perquè pugui rebre la paraula a cercar com a argument de línia d'ordres (args[0]). Si no en rep cap, utilitzarà ERROR per defecte.
Requisits de Comunicació en Paral·lel:
El pare ha d'enviar el mateix text d'entrada a tots dos processos fills de forma concurrent.
Se n'han de llegir els resultats evitant bloquejos mutus (Buffer Deadlock).
Format Estricte de Sortida: Per permetre l'avaluació automàtica, l'última línia que ha d'imprimir AnalitzadorPrincipal per consola ha de seguir exactament aquest format:

RESULTAT: ERRORS=3 | WARNINGS=5 | EXIT_CODE=0






Criteris de qualificació


S’ha d’entregar UN SOL FITXER PDF amb el vostre nom complet. Aquest fitxer ha de contenir un enllaç al codi font (Repositori Git o drive) i un enllaç a al vídeo (Drive o YouTube) recorda fer-lo d’una durada màxima de 3 minuts (180 segons).

Es qualificarà amb una nota de 0 a 10 amb la següent rúbrica.
Rubrica_PF1

Contingut obligatori:
Execució del Test (1 min): Mostrar la pantalla de l'IDE executant la classe AvaluadorPracticaTest i veure com passa els 3 nivells en verd.
Explicació Tècnica (2 min): Mostrar breument el codi font i explicar:
Com han gestionat la comunicació pare-fill a AnalitzadorPrincipal.
Com han evitat el bloqueig de buffer (Deadlock) o com han gestionat el flush().
On es comprova el codi de sortida (exitValue()) i la redirecció a errors_filtre.log.
Plantilla test: https://drive.google.com/file/d/1MgfEBslqurg-9x36l0PliD27XhfNz3xi/view?usp=sharing

Penalitzacions
En cas de plagi, errors de compilació o execució incorrecta, la pràctica serà qualificada amb 0 punts.
L’incompliment dels requisits de l’enunciat pot reduir parcialment la nota segons la gravetat.

