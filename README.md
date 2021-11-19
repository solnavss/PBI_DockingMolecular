# PBI_DockingMolecular

**Cinvestav Unidad Irapuato**

**Posgrado en Biología Integrativa**

**Química Biológica**

**Práctica - Docking con Autodock**

**Integrantes: Gabriel Osuna Osuna, Marisol Navarro Miranda y Pablo Adolfo Silva Villatoro**

**Fecha de entrega: 18/11/21**

**Información sobre el repositorio:** En este repositorio se encuentran todos los archivos generados a partir de realizar el [Taller de Simulación Molecular: Docking con Autodock 4.2](https://jriccil.github.io/Taller_Simulacion_Molecular/docking_con_adt4.html).


![alt text](https://github.com/solnavss/PBI_DockingMolecular/blob/main/imgs/img3.png)

```
#############################################################################
# INSTALACION DE AMBIENTES USANDO CONDA
#############################################################################

AMBIENTE_1
conda create -n ad4
conda activate ad4
conda install -c hcc autodock
conda install -c bioconda mgltools
conda deactivate

AMBIENTE_2
conda create -n dock
conda activate dock
conda install -c conda-forge pdb2pqr openbabel
conda install -c bioconda smina autodock-vina
conda deactivate

#############################################################################
# Docking con Autodock 4.2
#############################################################################
 
# Creamos un directorio de trabajo
mkdir wd_dk
cd wd_dk

# Descargar la proteina desde UCSF Chimera (ambiente gráfico)

# Activar el ambiente 2
conda activate dock

pdb2pqr30 --ff='AMBER' --ffout='AMBER' --with-ph=7.0 --drop-water --keep-chain --pdb-output prot.pdb prot_unprep.pdb pqr_file.pqr

# ////
(dock) -bash-4.2$ ls
pqr_file.log  pqr_file.pqr  prot.pdb  prot_unprep.pdb
# ////

# Convertir smiles a mol2
obabel -ismi smiles_atp.smi -omol2 -O ATP.mol2 --gen3d -p 7 --partialcharge gasteiger

# Desactivar ambiente dock
conda deactivate

#############################################################################
# Preparación de los archivos necesarios para Docking (PDBQT)
#############################################################################

# Activar el ambiente 1
conda activate ad4

# Genera el archivo PDBQT del ligando
prepare_ligand4.py -l ATP.mol2 -v -o ATP.pdbqt -d ligand_dict.py -U 'nphs_lps' -C

# Inspeccionar ligand_dict.py
cat ligand_dict.py

#############################################################################
Preparación del receptor en formato PDBQT
#############################################################################

prepare_receptor4.py -r prot.pdb -o prot.pdbqt -U 'nphs' -v

#############################################################################
Ejecución de Autogrid
#############################################################################

# Determinar el centro y tamaño de la rejilla
adt prot.pdbqt

# Crear el archivo GPF
prepare_gpf4.py -r prot.pdbqt -l ATP.pdbqt -d ligand_dict.py -p npts='66,52,60' -p ligand_types='A,C,HD,N,NA,OA,P' -p gridcenter='-80,-46,10' -o GPF.gpf

# Ejecutar Autogrid
autogrid4 -p GPF.gpf -l GLG.glg

# ////
(ad4) -bash-4.2$ ls
ATP.mol2   ligand_dict.py   prot.A.map   prot.NA.map  prot.e.map     prot.pdbqt
ATP.pdbqt  pqr_file.log     prot.C.map   prot.OA.map  prot.maps.fld  prot_unprep.pdb
GLG.glg    pqr_file.pqr     prot.HD.map  prot.P.map   prot.maps.xyz  smiles_atp.smi
GPF.gpf    prepare_gpf4.py  prot.N.map   prot.d.map   prot.pdb
# ////

# Obtencion del DPF.dpf

prepare_dpf42.py -l ATP.pdbqt -r prot.pdbqt -o DPF.dpf -p ga_num_evals='1000000' -p ga_run='3' -p ga_num_generations='27000' -p ga_pop_size='150' -p unbound_model='bound' -p rmstol='2.0' -p outlev='adt' -v -s

# Observacion: hay un error en el diagrama que se presenta para este paso.
cat DPF.dpf

#############################################################################
Extraer un resumen de los resultados
#############################################################################

summarize_results4.py -d . -r prot.pdbqt -b -e -k -o ATP_dock_results.txt

cat ATP_dock_results.txt

# ////
lowestEnergy_dlgfn #runs #cl #LEC LE rmsd_LE #hb #ESTAT #HB #VDW #DSOLV #ats #tors #h_ats #lig_eff
./DLG,   3,  3,  1,  4.4600, 81.4848, 0, -0.0129,  0.0000,  0.0000,  0.0000  39,15, 8,  0.1439
# ////

#############################################################################
Extrae la pose con la mejor afinidad (menor energía)
#############################################################################

write_lowest_energy_ligand.py -f DLG.dlg -o best_pose_ATP.pdbqt -N 
# Observacion: hay un typo en la linea y realmente es -f DLG.dlg

# Pose con mejor afinidad
pdbqt_to_pdb.py -f best_pose_ATP.pdbqt -o best_pose_ATP.pdb

# Desactivar ambiente dock
conda deactivate

# RUTA DE LOS SCRIPTS USADOS CON ad4
/home/sol/.conda/envs/ad4/MGLToolsPckgs/AutoDockTools/Docking.py
``` 
