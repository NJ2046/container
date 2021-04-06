# 概论
kubeflow是一个胶水项目，它把诸多对机器学习的支持，比如模型训练，超参数训练，模型部署等进行组合并已容器化的方式进行部署，提供整个流程各个系统的高可用及方便的进行扩展部署了 kubeflow的用户就可以利用它进行不同的机器学习任务。
### Pipelines使用
pipelines是 Kubeflow 社区新近开源的端到端的 ML/DL 工作流系统。实现了一个工作流模型。所谓工作流，或者称之为流水线（pipeline），可以将其当做一个有向无环图（DAG）。其中的每一个节点，在 kubeflow/pipelines 的语义下被称作组件（component）。组件在图中作为一个节点，其会处理真正的逻辑，比如预处理，数据清洗，模型训练等等。每一个组件负责的功能不同，但有一个共同点，即组件都是以 Docker 镜像的方式被打包，以容器的方式被运行的。
1. 准备一段 Traning 的代码
```
from __future__ import absolute_import, division, print_function, unicode_literals
import tensorflow as tf

def train_model():
  mnist = tf.keras.datasets.mnist
  # 下载 mnist 数据集，可以离线提前下载后填写路径
  # 不填就要走公网下载数据了，数据量不大，十多兆
  (x_train, y_train), (x_test, y_test) = mnist.load_data()
  x_train, x_test = x_train / 255.0, x_test / 255.0

  model = tf.keras.models.Sequential([
    tf.keras.layers.Flatten(input_shape=(28, 28)),
    tf.keras.layers.Dense(128, activation='relu'),
    tf.keras.layers.Dropout(0.2),
    tf.keras.layers.Dense(10, activation='softmax')
  ])
  model.compile(optimizer='adam',
                loss='sparse_categorical_crossentropy',
                metrics=['accuracy'])
  model.fit(x_train, y_train, epochs=5)
  model.evaluate(x_test, y_test)

if __name__ == '__main__':
  train_model()
```
2. 要把这个训练过程，放在Pipeline上运行，需要通过Python SDK将代码封装成一个component，然后再构建pipeline。
安装Python SDK
```
pip3 install http://kubeflow.oss-cn-beijing.aliyuncs.com/kfp/0.1.14/kfp.tar.gz --upgrade
pip3 install http://kubeflow.oss-cn-beijing.aliyuncs.com/kfp-arena/kfp-arena-0.4.tar.gz --upgrade
```
通过下面的Dockerfile将Traning的代码及数据集打包成镜像。
```
FROM tensorflow/tensorflow:1.14.0-py3
ADD mnist.py .
ENTRYPOINT ["python", "mnist.py"]
```
构建pipeline代码，并转成 .zip文件
```
import kfp
from kfp import dsl

def train_mnist_op():
  return dsl.ContainerOp(name='mnist', image='registry.cn-hangzhou.aliyuncs.com/nhj_images/mnist:v0.0.1')#这里是我生成的image

@dsl.pipeline(
    name='mnist train',
    description='test'
)
def pipeline():
  op = train_mnist_op()

if __name__ == '__main__':
  kfp.compiler.Compiler().compile(pipeline, __file__ + '.zip')
```
运行 python3  mnist_op.py生成zip文件(里面是yaml文件)

3.将生成的zip文件通过 Pipeline UI 上传（http://172.17.7.152:8088/_/pipeline/?ns=zykj），选择实验的环境，创建 Run。在实验模块的运行环境中，点击创建的Run，并查看结果。

参考文档链接：https://zhuanlan.zhihu.com/p/77607425
https://help.aliyun.com/document_detail/156212.html