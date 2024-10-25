# SAM Exporter - Now with Segment Anything 2!~~

Exporting [Segment Anything](https://github.com/facebookresearch/segment-anything), [MobileSAM](https://github.com/ChaoningZhang/MobileSAM), and [Segment Anything 2](https://github.com/facebookresearch/segment-anything-2) into ONNX format for easy deployment.

[![PyPI version](https://badge.fury.io/py/samexporter.svg)](https://badge.fury.io/py/samexporter)
[![Downloads](https://pepy.tech/badge/samexporter)](https://pepy.tech/project/samexporter)
[![Downloads](https://pepy.tech/badge/samexporter/month)](https://pepy.tech/project/samexporter)
[![Downloads](https://pepy.tech/badge/samexporter/week)](https://pepy.tech/project/samexporter)

**Supported models:**

- Segment Anything 2 (Tiny, Small, Base, Large) - **Note:** Experimental. Only image input is supported for now.
- Segment Anything (SAM ViT-B, SAM ViT-L, SAM ViT-H)
- MobileSAM

## Installation

Requirements:

- Python 3.10+

From PyPi:

```bash
pip install torch==2.4.0 torchvision --index-url https://download.pytorch.org/whl/cpu
pip install samexporter
```

From source:

```bash
pip install torch==2.4.0 torchvision --index-url https://download.pytorch.org/whl/cpu
git clone https://github.com/vietanhdev/samexporter
cd samexporter
pip install -e .
```

## Convert Segment Anything, MobileSAM to ONNX

- Download Segment Anything from [https://github.com/facebookresearch/segment-anything](https://github.com/facebookresearch/segment-anything).
- Download MobileSAM from [https://github.com/ChaoningZhang/MobileSAM](https://github.com/ChaoningZhang/MobileSAM).

```text
original_models
   + sam_vit_b_01ec64.pth
   + sam_vit_h_4b8939.pth
   + sam_vit_l_0b3195.pth
   + mobile_sam.pt
   ...
```

- Convert encoder SAM-H to ONNX format:

```bash
python -m samexporter.export_encoder --checkpoint original_models/sam_vit_h_4b8939.pth \
    --output output_models/sam_vit_h_4b8939.encoder.onnx \
    --model-type vit_h \
    --quantize-out output_models/sam_vit_h_4b8939.encoder.quant.onnx \
    --use-preprocess
```

- Convert decoder SAM-H to ONNX format:

```bash
python -m samexporter.export_decoder --checkpoint original_models/sam_vit_h_4b8939.pth \
    --output output_models/sam_vit_h_4b8939.decoder.onnx \
    --model-type vit_h \
    --quantize-out output_models/sam_vit_h_4b8939.decoder.quant.onnx \
    --return-single-mask
```

Remove `--return-single-mask` if you want to return multiple masks.

- Inference using the exported ONNX model:

```bash
python -m samexporter.inference \
    --encoder_model output_models/sam_vit_h_4b8939.encoder.onnx \
    --decoder_model output_models/sam_vit_h_4b8939.decoder.onnx \
    --image images/truck.jpg \
    --prompt images/truck_prompt.json \
    --output output_images/truck.png \
    --show
```

![truck](https://raw.githubusercontent.com/vietanhdev/samexporter/main/sample_outputs/truck.png)

```bash
python -m samexporter.inference \
    --encoder_model output_models/sam_vit_h_4b8939.encoder.onnx \
    --decoder_model output_models/sam_vit_h_4b8939.decoder.onnx \
    --image images/plants.png \
    --prompt images/plants_prompt1.json \
    --output output_images/plants_01.png \
    --show
```

![plants_01](https://raw.githubusercontent.com/vietanhdev/samexporter/main/sample_outputs/plants_01.png)

```bash
python -m samexporter.inference \
    --encoder_model output_models/sam_vit_h_4b8939.encoder.onnx \
    --decoder_model output_models/sam_vit_h_4b8939.decoder.onnx \
    --image images/plants.png \
    --prompt images/plants_prompt2.json \
    --output output_images/plants_02.png \
    --show
```

![plants_02](https://raw.githubusercontent.com/vietanhdev/samexporter/main/sample_outputs/plants_02.png)


**Short options:**

- Convert all Segment Anything models to ONNX format:

```bash
bash convert_all_meta_sam.sh
```

- Convert MobileSAM to ONNX format:

```bash
bash convert_mobile_sam.sh
```

## Convert Segment Anything 2 to ONNX

### INSTALLATION
```bash
pip install git+https://github.com/facebookresearch/sam2.git@c2ec8e14a185632b0a5d8b161928ceb50197eddc
pip install onnx
python samexporter/export_sam21_cvat.py \
      --checkpoint=original_models/sam2.1_hiera_large.pt \
      --output_encoder output_models/sam2.1_hiera_large.encoder.onnx \
      --output_decoder output_models/sam2.1_hiera_large.decoder.onnx \
      --model_type sam2.1_hiera_large
```
NOTE: Might cause OOM on CPU 

- Download Segment Anything 2 from [https://github.com/facebookresearch/segment-anything-2.git](https://github.com/facebookresearch/segment-anything-2.git). You can do it by:

```bash
cd original_models
bash download_sam2.sh
```

The models will be downloaded to the `original_models` folder:

```text
original_models
    + sam2_hiera_tiny.pt
    + sam2_hiera_small.pt
    + sam2_hiera_base_plus.pt
    + sam2_hiera_large.pt
   ...
```

- Install dependencies:

```bash
pip install git+https://github.com/facebookresearch/segment-anything-2.git
```

- Convert all Segment Anything models to ONNX format:

```bash
bash convert_all_meta_sam2.sh
```

- Inference using the exported ONNX model (only image input is supported for now):

```bash
python -m samexporter.inference \
    --encoder_model output_models/sam2_hiera_tiny.encoder.onnx \
    --decoder_model output_models/sam2_hiera_tiny.decoder.onnx \
    --image images/plants.png \
    --prompt images/truck_prompt_2.json \
    --output output_images/plants_prompt_2_sam2.png \
    --sam_variant sam2 \
    --show
```

![truck_sam2](https://raw.githubusercontent.com/vietanhdev/samexporter/main/sample_outputs/sam2_truck.png)

## Tips

- Use "quantized" models for faster inference and smaller model size. However, the accuracy may be lower than the original models.
- SAM-B is the most lightweight model, but it has the lowest accuracy. SAM-H is the most accurate model, but it has the largest model size. SAM-M is a good trade-off between accuracy and model size.

## AnyLabeling

This package was originally developed for auto labeling feature in [AnyLabeling](https://github.com/vietanhdev/anylabeling) project. However, you can use it for other purposes.

[![AnyLabeling](https://user-images.githubusercontent.com/18329471/236625792-07f01838-3f69-48b0-a12e-30bad27bd921.gif)](https://youtu.be/5qVJiYNX5Kk)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## References

- ONNX-SAM2-Segment-Anything: [https://github.com/ibaiGorordo/ONNX-SAM2-Segment-Anything](https://github.com/ibaiGorordo/ONNX-SAM2-Segment-Anything).
