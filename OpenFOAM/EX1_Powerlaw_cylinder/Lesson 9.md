![[Pasted image 20230429164948.png]]


sempre sti file vuole, entriamo in constant, oltre a polymesh ci sono due file che prima non ci stavano e adesso vuole,
in momentum transport

simulationType laminar;

poi physical properties


nu è viscosità cinematica , il solver vuole eta su rho già perché useremo un sovlver per fluidi incomprimibile, 
vsle 0.1 , la roba tra quadre sono 7 numeri , servono per le unità di misura
sono gli esponenti delle unità di misura della nu
kg m secondi 

gli altri sono gradi kelvin gli altre 3 sono cose di elettromagnetismo

in 0 ci sono p e U

in 0 ci sono le boundary e le initial conditions

apriamo U
internalField sarebbe initial values di comsol o la condizione iniziale o la lil primo tentativo per lo stazionario 

boundaryField 

inflow che è il nome dato su blockMeshDict 
type ... wall, symmetryPlane, patch(in e out), wedge sono 4 possibili condizioni al contorno 
 che abbiamo specificato in blockmesh dict
 in U fixedValue 
value uniform(vettore) 

il profilo parabolico non posso usare l'equazione, o la scrivo in cpp sapendo la y come si chiama in openfoam oppure ci sono dei toolbox che consentono di inserire profili variabili 

outflow zerogradient ho la faccia annulla il gradiente della velocità normale alla faccia sarebbe n scalar gradiente di velocità uguale 0

noSlip sarebbe come inflow ma le componenti sono 0

wedge spicchio dell'altra volta fa sarebbero le facce per capire che se ruoto fanno la geoemtria completa

in comsol non abbiamo mai usato condizioni su p, ma il solver pisoFoam fa uscire delle equazioni in cui esce come incognita la p per cui gli serve la condizioni al contorno, escono due equazioni una in p una in v e poi va ricorsivamente, quindi dobbiamo pensare anche a quale condizioni sono più opportune

inflow conosco il dp magari ma se specifico il la velocità non posso specificarla, se metto una velocità in ingresso devo mettere zero gradient ovvero la pressione è costante all'ingresso dp/dx è zero prima del dominio
zero gradient pure sul wall ovvero l'isolinia di pressione è ortogonale alla parete sul alla aprete la pressione rispetta la condizione di zero gradient 

in openfoam non si parte da zero, si parte da un tutorial e poi si modifica 

ogni solver per fare cose diverse vedi guida

fv sarebbero finite volume sarebbe cambiare i parametri del solver
 non mettere mai mano

nNonOrthogonalityCorrectors 0; se la mesh è totalmente ortoganle va bene sennò sono un po' storti e checkmesh ti dice qualcosa allora si cambia di solito a 2 così cambi un po' pisofoam fa più giri e diventa più lento ma l'errore dovuto alla non ortogonalità diminuisce 

controlDict in system

dobbiamo scegliere dT
quindi dovremo andare a tentativi 
