Il file @cambiamenti.txt contiene i cambiamenti della specifica A2A tra la versione 0.2.5         
attualmente implementata nel TCK contenuto nella directory corrente). Fai un clone temporaneo del 
repository e tag https://github.com/a2aproject/A2A/tree/v0.2.6. COncentrandoti sulle directory    
types, specification ed il file doc/specification.md analizza cosa serve per implementare nel     
progetto corrente (che è un TCK di quella specifica) le novità e renderlo utile per testare       
sistemi che imlementino A2A 0.2.6. Il tuo compito è quello di impersonare un architetto software  
esperto della specifica A2A e scrivere un piano di sviluppo dettagliato per un programmatore      
junior che implementerà quanto richiesto. Sii preciso e speifico, organizza il piano per fasi,    
task e sub-task successivi in modo che sia possibile interrompere il lavoro e riprenderlo in un   
secondo momento. Chiedi al programmatore di tenere traccia del lavoro in un file chiamato         
working_progress.md da aggiornare al termine di ogni fase. Richiedi frequenti checkpoint con      
commit git che abbiano commenti sintetici ma chiari del lavoro fatto. Richiedi che al termine di  
ogni fase sia verificato che il risultato dei test su un SUT già avviato dal coordinatore del     
progetto non venga peggiorato (risultati su test preesistenti identico). Il piano deve includere  
anche l'adattamento degli script quali @run_tck.py e @util_scripts/generate_compliance_report.py  
per considerare i diversi trasporti agginti in spec 0.2.6. La buona riuscita del progetto dipende 
da una file di specifica da te generato preciso e chiaro per il programmatore junior. Salva il tuo
piano in un file chiamato porting_026.md



Il file @cambiamenti.txt contiene i cambiamenti della specifica A2A tra la versione 0.2.5         
(attualmente implementata nel TCK contenuto nella directory corrente). Fai un clone temporaneo del
repository e tag https://github.com/a2aproject/A2A/tree/v0.2.6. COncentrandoti sulle directory    
types, specification ed il file doc/specification.md analizza cosa serve per implementare nel     
progetto corrente (che è un TCK di quella specifica) le novità e renderlo utile per testare       
sistemi che imlementino A2A 0.2.6. Il tuo compito è quello di impersonare un architetto software  
esperto della specifica A2A e scrivere un piano di sviluppo dettagliato per un programmatore      
junior che implementerà quanto richiesto. Sii preciso e speifico, organizza il piano per fasi,    
task e sub-task successivi in modo che sia possibile interrompere il lavoro e riprenderlo in un   
secondo momento. Chiedi al programmatore di tenere traccia del lavoro in un file chiamato         
working_progress.md da aggiornare al termine di ogni fase. Richiedi frequenti checkpoint con      
commit git che abbiano commenti sintetici ma chiari del lavoro fatto. Richiedi che al termine di  
ogni fase sia verificato che il risultato dei test su un SUT già avviato dal coordinatore del     
progetto non venga peggiorato (risultati su test preesistenti identico). Il piano deve includere  
anche l'adattamento degli script quali @run_tck.py e @util_scripts/generate_compliance_report.py  
per considerare i diversi trasporti agginti in spec 0.2.6. La buona riuscita del progetto dipende 
da una file di specifica da te generato preciso e chiaro per il programmatore junior. Salva il tuo
piano in un file chiamato porting_026.md



Ho guardto bene il documento d ate generato. Non vedo istruzioni dettagliate per un Junior developer per scaricarsi la versione corretta della          
specifica. Credo lo dovremmo aggiungere, e anche spiegare al developer su quali file concentrarsi. Rimarca anche che la specifica è l'unica sorgente di 
verità e che ogni implementazione deve essere validata da quanto contenuto in specifica e che i test scritti dovranno contenere docstring che facciano  
riferimento al paragrafo della specifica per il quale il test è stato implentato (come già fatto per JSONRPC). CHiarisci molto bene che tutti i test,   
il codice, i commenti e i docstring dovranno esser ein inglese. La lingua italiana è usata solo ed unicamente per il documento di specifica. Aggiorna   
@porting_026.md