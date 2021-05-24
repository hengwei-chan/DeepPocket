# DeepPocket

DeepPocket is a 3D convolutional Neural Network framework for ligand binding site detection and segmentation from protein structures. This is the official open source repository for the following paper:

Aggarwal, Rishal; Gupta, Akash; Chelur, Vineeth; Jawahar, C. V.; Priyakumar, U. Deva (2021): DeepPocket: Ligand Binding Site Detection and Segmentation using 3D Convolutional Neural Networks. ChemRxiv. Preprint. https://doi.org/10.26434/chemrxiv.14611146.v1 

## Requirements

[Fpocket](https://github.com/Discngine/fpocket), [Pytorch](https://pytorch.org/), [libmolgrid](https://github.com/gnina/libmolgrid), [Biopython](https://biopython.org/) and other frequently used python packages

## Dataset Preprocessing

PDB files are first paresed to remove hetero atoms, then converted to "gninatypes" files and finally collected into a "molcache" file for quicker input and model training with libmolgrid. More about "gninatypes" and "molcache" [here](https://github.com/gnina/gnina). 

cavity6.mol2 files that are provided by scPDB and generated by volsite for other datasets are used as is, the "data_dir" argument in training scripts have to be pointed to the parent directory they are present.

".types" files contain training data points prepared, the first column is the class label, the next three columns are pocket center cordinates (x,y,z) and the final columns contain molecule files required for that datapoint. All the molecule files specified in the types files must be present in either the molcache or in the "data_dir". 

Prepared types, molcache and saved model checkpoints can be downloaded here.

I know this may be very cryptic, therefore I have written down simple steps in the last section of the README that one can use to prepare a new dataset for training. 

## Predicting Binding Site

"predict.py" is a simple script that can be used for predicting binding sites from a .pdb file. It follows 6 steps namely:
1) Hetero atom removal (clean_pdb)
2) fpocket run
3) Parsing fpocket output for candidate centers (get_centers)
4) Creating gninatypes and types file for CNN input (types_and_gninatyper)
5) Rerank types input according to CNN score (rank_pockets)
6) Segment shape of top ranked pockets (segment_pockets)

Example usage of predict.py:

    python predict.py -p protein.pdb -c first_model_fold1_best_test_auc_85001.pth.tar -s seg0_best_test_IOU_91.pth.tar -r 3

Description of each argument given in script.
If the name of the input file is protein.pdb, then fpocket creates a protein_out/pockets directory. The CNN ranked pockets will be given in the bary_centers_ranked.types file in that directory. If you asked for segmented pockets ("-r") the script will output ".dx" files that can be visualised in pymol. 

## Training Classifier

We use [wandb](https://wandb.ai/site) to track training performance. It's free and easy to use. If you want to avoid using wandb, simply comment out all lines that contain "wandb" in the training script.

Example usage of train.py:

    python train.py -m model.py --train_types scPDB_train0.types --test_types scPDB_test0.types -i 200000 --train_recmolcache scPDB_new.molcache2 --test_recmolcache scPDB_new.molcache2 -r val0 -o /model_saves/val9 --base_lr 0.001 --solver Adam 

Description of each argument given in script.

## Training segmentation

Example usage of train_segmentation.py:

    python train_segmentation.py --train_types seg_scPDB_train9.types --test_types seg_scPDB_test9.types -d data/ --train_recmolcache scPDB_new.molcache2 --test_recmolcache scPDB_new.molcache2 -b 8 -o model_saves/seg9 -e 200 -r seg9
    
Description of each argument in script

## Preparing Data
