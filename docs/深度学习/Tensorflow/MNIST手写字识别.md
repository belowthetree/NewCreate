>先声明：本人前期学习借助黄文坚的《Tensorflow深度学习实战》（应该是这个名字）

>本篇是针对有一定基础的，没基础的去我之前的教程里学习
## 第一步：先封装各种需要的功能
### 变量初始化函数

```
def weight_init(shape):
    return tf.Variable(tf.truncated_normal(shape, stddev = 0.1))
```
这是一个权重初始化函数，权重shape是一个多维数组

`tf.truncated_normal(shape, stddev = stddev)`采用了正太分布给变量赋值，根据实践经验，这种方式能够
较大程度提升拟合速度。

```
def biase_init(shape):
    return tf.Variable(tf.constant(0.1,shape=shape))
```
对于偏置的初始值我们可以随意一点

### 封装一下卷积、池化函数
```
def conv2d(x,w):
    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding="SAME")
```
x, w分别是图片和权重，将图片传进来后用权重w进行卷积运算，采用'SAME'是为了不改变图片大小

```
def max_pool_2x2(x):
    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding="SAME")
```
池化函数同理

### 接下来构建网络结构
```
x = tf.placeholder(tf.float32,[None,784])
y = tf.placeholder(tf.float32,[None,10])
x_images = tf.reshape(x,[-1,28,28,1])

w1_conv = weight_init([5,5,1,32])
b1_conv = biase_init([32])
h1_conv = tf.nn.relu(conv2d(x_images,w1_conv) + b1_conv)
h1_pool = max_pool_2x2(h1_conv)

w2_conv = weight_init([5,5,32,64])
b2_conv = biase_init([64])
h2_conv = tf.nn.relu(conv2d(h1_pool,w2_conv) + b2_conv)
h2_pool = max_pool_2x2(h2_conv)
```
>这里是卷积部分

这里采用的是5X5大小的卷积，特征值32->64，激活函数采用relu。对于比较简单的图片，不需要输出很多的特征值
，也不需要太深的网络。图片这时经过了两次池化从28X28变成了7X7，总的数据量是7X7X64
```
w1_fc = weight_init([7*7*64,1024])
b1_fc = biase_init([1024])
h2_pool_flat = tf.reshape(h2_pool,[-1,7*7*64])
h1_fc = tf.nn.relu(tf.matmul(h2_pool_flat,w1_fc) + b1_fc)
```
>这里是线性部分

这里先利用`tf.reshape`将图片数据由多维数组变成一维数组。这里采用了一个矩阵乘法，用了这个大小的矩阵`[7X7X64，1024]`。
利用的是矩阵的映射功能（别的篇章会解释）。

```
import tensorflow as tf
import tensorflow.examples.tutorials.mnist.input_data as data
mnist = data.read_data_sets("data/fashion",one_hot = True)
sess = tf.InteractiveSession()

def weight_init(shape):
    return tf.Variable(tf.truncated_normal(shape, stddev = 0.1))

def biase_init(shape):
    #we use a small possitive number to initialize biases
    return tf.Variable(tf.constant(0.1,shape=shape))

def conv2d(x,w):
    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding="SAME")

def max_pool_2x2(x):
    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding="SAME")



x = tf.placeholder(tf.float32,[None,784])
y = tf.placeholder(tf.float32,[None,10])
x_images = tf.reshape(x,[-1,28,28,1])

w1_conv = weight_init([5,5,1,32])
b1_conv = biase_init([32])
h1_conv = tf.nn.relu(conv2d(x_images,w1_conv) + b1_conv)
h1_pool = max_pool_2x2(h1_conv)

w2_conv = weight_init([5,5,32,64])
b2_conv = biase_init([64])
h2_conv = tf.nn.relu(conv2d(h1_pool,w2_conv) + b2_conv)
h2_pool = max_pool_2x2(h2_conv)


w1_fc = weight_init([7*7*64,1024])
b1_fc = biase_init([1024])
h2_pool_flat = tf.reshape(h2_pool,[-1,7*7*64])
h1_fc = tf.nn.relu(tf.matmul(h2_pool_flat,w1_fc) + b1_fc)


keep_prob = tf.placeholder(tf.float32)
drop_out = tf.nn.dropout(h1_fc,keep_prob)

w2_fc = weight_init([1024,10])
b2_fc = biase_init([10])
y_predict = tf.nn.softmax(-tf.matmul(drop_out,w2_fc) + b2_fc)

cross_entropy = tf.reduce_mean(-tf.reduce_sum(y*tf.log(y_predict),reduction_indices=[1]))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

correct_prediction = tf.equal(tf.argmax(y_predict,1),tf.argmax(y,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32))

writer=tf.summary.FileWriter(r"./tensorboard",tf.get_default_graph())
saver = tf.train.Saver()
tf.summary.scalar('accuracy',accuracy)
tf.summary.scalar('loss',loss)

tf.global_variables_initializer().run()
saver = tf.train.Saver()

for i in range(20000):
    batch = mnist.train.next_batch(50)
    if i%100 == 0:
        x_batch, y_batch = mnist.test.next_batch(100)
        train_accuracy = accuracy.eval(feed_dict = {x:x_batch,y:y_batch,keep_prob:1.0})
        print("train step %d, accuracy is : %g"%(i,train_accuracy))
        saver.save(sess, './tensorboard')

    train_step.run(feed_dict = {x:batch[0],y:batch[1],keep_prob:0.5})

#print("accuracy is %g"%accuracy.eval(feed_dict = {x:mnist.test.images,y:mnist.test.labels,keep_prob:1.0}))
#attention!!!
#if we use full test data to figure out the rate, it will probably exceed system memory
#batch = mnist.test.next_batch(5000)
#print("accuracy is %g"%accuracy.eval(feed_dict = {x:batch[0],y:batch[1],keep_prob:1.0}))

x_batch, y_batch = mnist.test.next_batch(100)
train_accuracy = accuracy.eval(feed_dict = {x:x_batch,y:y_batch,keep_prob:1.0})
print("train step %d, accuracy is : %g"%(i,train_accuracy))

saver.save(sess,"./model/normalCNN/model.ckpt")
sum = 0.0
for i in range(500):
    t = mnist.test.next_batch(100)
    a = accuracy.eval({x:t[0],y:t[1],keep_prob:1.0})
    sum += a
    print("result is :%g"%a)
print("average accuracy is :%g"%(sum/500))
```