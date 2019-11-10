# High Performance Motion Detector

You can download the video clip which is used in the demo at: https://box.bw-sw.com/f/c629c692d5c04b7caac6/?dl

Place it in the `tmp` directory.

Algorithm demonstration video screencast can be found at: https://youtu.be/FCme11alEmc

## Detector usage and parameters

* `bg_subs_scale_percent` &ndash; how much to scale initial frame before movement detection occurs (default: **1/4**);
* `bg_history` &ndash; the length of background accumulator ring buffer (default: **15**);
* `bg_history_collection_period_max` &ndash; defines how often update background ring buffer with frames from movement (default: **1** &ndash; every frame);
* `movement_frames_history` &ndash; how much frames to keep in movement accumulator ring buffer (default: **5**);
* `brightness_discard_level` &ndash; threshold which is used to detect movement from the noise (default: **20**);
* `pixel_compression_ratio` &ndash; how much to compress the initial video for boxes search (default: **0.1**), means that every **10x10 px** of initial frame will be resized to **1x1 px** of detection frame;
* `group_boxes` &ndash; group overlapping boxes into a single one or just keep them as they are (default: **True**);
* `expansion_step` &ndash; how big is expansion algoritm step when it searches for boxes, lower steps lead to smaller performance and close objects are detected as separate, bigger step leads to faster algorithm performance and close objects can be detected as a single one (default: **1**).

```python
import cv2
import numpy as np
from time import time
from detector import MovementDetector

if __name__ == "__main__":

    cap = cv2.VideoCapture('tmp/helmets-v1-55.mp4')

    # cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1920)
    # cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 1080)

    detector = MovementDetector(bg_history=20,
                                expansion_step=5)

    res = []
    while True:
        # Capture frame-by-frame
        ret, frame = cap.read()
        begin = time()
        boxes = detector.detect(frame)
        end = time()
        res.append(1000 * (end - begin))
        print("StdDev: %.4f" % np.std(res), "Mean: %.4f" % np.mean(res), "Boxes found: ", len(boxes))
        cv2.imshow('last_frame', detector.frame)
        cv2.imshow('detect_frame', detector.detection)
        cv2.imshow('diff_frame', detector.color_movement)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
```

## Performance

The performance depends **a lot** on the next factors:

* Background substraction scale [`bg_subs_scale_percent`] (default 1/4), which leads to 480x230 frame for initial 1480x920 frame.
* Size of the frame which is used to search for bounding boxes [`pixel_compression_ratio`] (default 1/10), which leads to 148x92 for initial 1480x920 frame.
* Expansion step [`expansion_step`] which is used to find bounding boxes.

So, for the sample [video](https://box.bw-sw.com/f/c629c692d5c04b7caac6/?dl) (1480x920@30FPS) and all these parameters set to default the expected performance results for a single frame processing are:

* Mean frame processing time is `8.3690 ms`
* Standard deviation is `9.3835 ms`

on `Intel(R) Core(TM) i5-7440HQ CPU @ 2.80GHz` CPU.

## Author

The code is developed by [Ivan Kudriavtsev](https://www.facebook.com/ivan.kudryavtsev.35) @ [Bitworks Software](https://bitworks.software/) &ndash; custom software development company focused on delivery of scalable, BigData & ML solutions for customers worldwide.

Find me on FB: https://www.facebook.com/ivan.kudryavtsev.35

![Bitworks Software](https://github.com/bwsw/cloudstack-ui/blob/master/screens/15047882.png)
