# Brief Report: Object Detection with YOLOv8

## 1. Summary of Key Findings

The evaluation of computer vision tasks highlights a progression in spatial awareness: image classification provides a single category label, object detection introduces spatial localization via bounding boxes, and instance segmentation maps exact pixel boundaries for individual objects. The YOLO (You Only Look Once) architecture achieves rapid inference by treating detection as a single regression problem, bypassing the slower Region Proposal Network (RPN) found in two-stage models like Faster R-CNN.

Model performance is mathematically validated using Intersection over Union (IoU). An IoU threshold of 0.50 is the standard baseline to classify a prediction as a True Positive. From a deployment perspective, fine-tuning a YOLO model on a custom dataset requires strict formatting, specifically normalizing all bounding box coordinates (x_center, y_center, width, height) to values between 0.0 and 1.0 relative to the image dimensions.

## 2. YOLO Variants: Speed vs. Accuracy Trade-offs

Testing the YOLOv8 architecture on the sample dataset revealed a strict inverse relationship between model depth (which correlates with accuracy/mAP) and inference speed (FPS). Based on the hardware benchmarking conducted in Module C, the metrics scale as follows:

* **YOLOv8n (Nano):** Achieved 16.14 FPS with an inference time of 61.96 ms. With only ~3.2 million parameters, this model is highly optimized for edge devices requiring real-time performance at the cost of precision on smaller objects.
* **YOLOv8s (Small):** Operated at 8.16 FPS with an inference time of 122.59 ms. This variant doubles the parameter count, offering a stronger baseline accuracy while remaining viable for standard video processing.
* **YOLOv8m (Medium):** Dropped to 4.14 FPS with an inference time of 241.77 ms. While this model captures highly complex features and achieves higher accuracy thresholds, it cannot sustain real-time video processing (≥ 30 FPS) without dedicated GPU acceleration.

## 3. Challenges in Object Detection

While static image detection is highly reliable, applying object detection to continuous video introduces distinct computational and environmental hurdles:

* **Temporal Instability (Flickering):** Because standard object detectors process every frame independently, slight variations in lighting or pixel shading can cause confidence scores to fluctuate above and below the threshold, resulting in bounding boxes rapidly blinking on and off.
* **Occlusion and Identity Loss:** Objects frequently move behind physical obstacles. When an object reappears, a baseline YOLO model treats it as an entirely new entity. Overcoming this requires integrating secondary Multi-Object Tracking (MOT) algorithms to maintain object permanence.
* **Motion Blur:** Fast movement degrades edge details within frames. Since convolutional neural networks rely heavily on edge and texture features, blurred frames significantly lower detection confidence and recall rates.

## 4. Detection Visualization

Below is the visualization of the standard sample image processed by the YOLOv8 model, demonstrating the model's ability to localize and classify multiple overlapping objects in a single forward pass.

**Inference Output Data:**
The model successfully filtered multiple predictions using Non-Maximum Suppression (NMS) to isolate six distinct objects. The primary subject (bus) was detected with the highest confidence, while partially occluded objects registered lower confidence scores.

| Class | Confidence Score | Bounding Box `[x1, y1, x2, y2]` |
| --- | --- | --- |
| bus | 0.87 (87.3%) | `[23, 231, 805, 757]` |
| person | 0.87 (86.5%) | `[49, 399, 245, 903]` |
| person | 0.85 (85.2%) | `[669, 392, 810, 877]` |
| person | 0.82 (82.5%) | `[222, 406, 345, 858]` |
| person | 0.26 (26.1%) | `[0, 551, 63, 873]` |
| stop sign | 0.25 (25.5%) | `[0, 254, 33, 325]` |