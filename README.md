UIDM-BREAST — RUNNING INSTRUCTIONS (WINDOWS)
============================================

1. Extract UIDM_Breast_Reproducible_Code.zip and open a terminal inside the extracted UIDM_Breast_Reproducible_Code folder.

2. Install dependencies (Python 3.10+):
   python -m pip install -r requirements.txt

3. Quick setup check:
   python -m uidm.cli smoke --out outputs/smoke

4. Download BUSI and organize the ultrasound images as:
   DATA_ROOT/normal/
   DATA_ROOT/benign/
   DATA_ROOT/malignant/
   (Replace DATA_ROOT below with the actual BUSI folder path.)

5. Prepare data and train the diffusion model:
   python -m uidm.cli prepare --busi DATA_ROOT --out runs/busi_manifest.csv
   python -m uidm.cli train-ddpm --out runs/ddpm

6. Generate diffusion-based features:
   python -m uidm.cli build-features --manifest runs/busi_manifest.csv --split train --ddpm runs/ddpm/ddpm_best.pt --out runs/features_uidm_train.npz
   python -m uidm.cli build-features --manifest runs/busi_manifest.csv --split val --ddpm runs/ddpm/ddpm_best.pt --out runs/features_uidm_val.npz

7. Train the classifiers:
   python -m uidm.cli train-classifier --manifest runs/busi_manifest.csv --mode raw --out runs/raw_cnn
   python -m uidm.cli train-classifier --manifest runs/busi_manifest.csv --mode uidm --train-cache runs/features_uidm_train.npz --val-cache runs/features_uidm_val.npz --out runs/uidm_cnn

8. Evaluate the selected models on the test split:
   python -m uidm.cli build-features --manifest runs/busi_manifest.csv --split test --ddpm runs/ddpm/ddpm_best.pt --out runs/features_uidm_test.npz
   python -m uidm.cli evaluate --manifest runs/busi_manifest.csv --split test --classifier runs/raw_cnn/classifier_best.pt --mode raw --out runs/results/raw
   python -m uidm.cli evaluate --manifest runs/busi_manifest.csv --split test --classifier runs/uidm_cnn/classifier_best.pt --mode uidm --cache runs/features_uidm_test.npz --out runs/results/uidm
   python -m uidm.cli compare --pred-a runs/results/raw/predictions.csv --pred-b runs/results/uidm/predictions.csv --out runs/results/paired_comparison.json

9. Find the final evaluation files under runs/results/.
