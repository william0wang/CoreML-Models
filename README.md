# CoreML-Models

## Convert AnimeGANv2 to Core ML

[AnimeGANv2](https://github.com/TachibanaYoshino/AnimeGANv2)

1. Save the graph in pbtxt format.

```python
   tf.train.write_graph(sess.graph_def, './', 'animegan.pbtxt')
```
2. Find the name of the output node.

```python
   graph = sess.graph
   print([node.name for node in graph.as_graph_def().node])
```

3. Make a frozen graph.

```python
   from tensorflow.python.tools.freeze_graph import freeze_graph
   import tfcoreml

   graph_def_file = 'animegan.pbtxt'
   checkpoint_file = 'checkpoint/generator_Hayao_weight/Hayao-64.ckpt'
   frozen_model_file = './frozen_model.pb'
   output_node_names = 'generator/G_MODEL/out_layer/Tanh'

   freeze_graph(input_graph=graph_def_file,
               input_saver="",
               input_binary=False,
               input_checkpoint=checkpoint_file,
               output_node_names=output_node_names,
               restore_op_name="save/restore_all",
               filename_tensor_name="save/Const:0",
               output_graph=frozen_model_file,
               clear_devices=True,
               initializer_nodes="")
```

4. Convert with tfcoreml.

```python
   input_tensor_shapes = {'test:0':[1, 256, 256, 3]} # batch size is 1
   # Output CoreML model path
   coreml_model_file = './animegan.mlmodel'
   output_tensor_names = ['generator/G_MODEL/out_layer/Tanh:0']
   # Call the converter
   coreml_model = tfcoreml.convert(
         tf_model_path='frozen_model.pb',
         mlmodel_path=coreml_model_file,
         input_name_shape_dict=input_tensor_shapes,
         output_feature_names=output_tensor_names,
         image_input_names='test:0',
         red_bias=-1,
         green_bias=-1,
         blue_bias=-1,
         image_scale=2/255,
         minimum_ios_deployment_target='12'
         )
```


## Converted CoreML Models

**Image Classifier**
| Google Drive Link | Size | Original Project |
| ------------- | ------------- | ------------- |
| [Efficientnetb0](https://drive.google.com/file/d/1mJq8SMuDaCQHW77ui3fAfe5o3Qu2GKMi/view?usp=sharing) | 22.7 MB | [TensorFlowHub](https://tfhub.dev/tensorflow/efficientnet/b0/classification/1)  |

**GAN**

| Google Drive Link | Size | Original Project |
| ------------- | ------------- | ------------- |
| [UGATIT_selfie2anime](https://drive.google.com/file/d/1cOB1comTnd5I22htZ4_OJ7tFQuQEI2ne/view?usp=sharing) | 1.12GB | [taki0112/UGATIT](https://github.com/taki0112/UGATIT)  |
| [AnimeGANv2_Hayao](https://drive.google.com/file/d/1i_kwj41BxA1xZNu2B7yX2VVqNF66atMN/view?usp=sharing)　| 8.7MB | [TachibanaYoshino/AnimeGANv2](https://github.com/TachibanaYoshino/AnimeGANv2)|
| [AnimeGANv2_Paprika](https://drive.google.com/file/d/1wuoaVoI8-HOOQ1kUiZkVJ9GWnPbtPnWF/view?usp=sharing)　| 8.7MB | [TachibanaYoshino/AnimeGANv2](https://github.com/TachibanaYoshino/AnimeGANv2)|
| [WarpGAN Caricature](https://drive.google.com/file/d/1QjfA6DSOu7Za1zY-b9ajX6tsIWe9-C16/view?usp=sharing)　| 35.5MB | [seasonSH/WarpGAN](https://github.com/seasonSH/WarpGAN)|
| [CartoonGAN_Shinkai](https://drive.google.com/file/d/1j9bvHFBX5yctSeaE8FEvUv-r-hEVvXwi/view?usp=sharing)　| 44.6MB | [mnicnc404/CartoonGan-tensorflow](https://github.com/mnicnc404/CartoonGan-tensorflow)|
| [CartoonGAN_Hayao](https://drive.google.com/file/d/1-2dTGge4fza-TTBI9actkg_xp91zYT-F/view?usp=sharing)　| 44.6MB | [mnicnc404/CartoonGan-tensorflow](https://github.com/mnicnc404/CartoonGan-tensorflow)|
| [CartoonGAN_Hosoda](https://drive.google.com/file/d/1-5VB1g7kRt0KMe6u37fi_t18l-Zn_wr1/view?usp=sharing)　| 44.6MB | [mnicnc404/CartoonGan-tensorflow](https://github.com/mnicnc404/CartoonGan-tensorflow)|
| [CartoonGAN_Paprika](https://drive.google.com/file/d/1-5x3TYugodcnGYiEEDitFqMQPVHsCDs_/view?usp=sharing)　| 44.6MB | [mnicnc404/CartoonGan-tensorflow](https://github.com/mnicnc404/CartoonGan-tensorflow)|

<img width="256" alt="スクリーンショット 2020-05-19 11 09 03" src="https://user-images.githubusercontent.com/23278992/85667417-881d7d00-b6f8-11ea-8d1c-0d66b2b72de9.png">
<!-- <img width="256" alt="スクリーンショット 2020-06-22 4 10 54" src="https://user-images.githubusercontent.com/23278992/85667453-91a6e500-b6f8-11ea-84bf-22853b0995dc.png"> -->

## How to use in a xcode project.

### 1,Use [CoreGANContainer](https://github.com/john-rocky/CoreGANContainer). You can use models with dragging&dropping into the container project. 

### 2,Or implement Vision request.

```swift:

import Vision
lazy var coreMLRequest:VNCoreMLRequest = {
   let model = try! VNCoreMLModel(for: modelname().model)
   let request = VNCoreMLRequest(model: model, completionHandler: self.coreMLCompletionHandler)
   return request
   }()

let handler = VNImageRequestHandler(ciImage: ciimage,options: [:])
   DispatchQueue.global(qos: .userInitiated).async {
   try? handler.perform([coreMLRequest])
}
```

For visualizing multiArray as image, Mr. Hollance’s “CoreML Helpers” are very convenient.
[CoreML Helpers](https://github.com/hollance/CoreMLHelpers)

[Converting from MultiArray to Image with CoreML Helpers.](https://medium.com/@rockyshikoku/converting-from-multiarray-to-image-with-coreml-helpers-59fdc34d80d8)

```swift:
func coreMLCompletionHandler（request：VNRequest？、error：Error？）{
   let = coreMLRequest.results？.first as！VNCoreMLFeatureValueObservation
   let multiArray = result.featureValue.multiArrayValue
   let cgimage = multiArray？.cgImage（min：-1、max：1、channel：nil）
```


Apps made by Core ML models.
[AnimateU](https://apps.apple.com/us/app/animateu/id1513582287)
