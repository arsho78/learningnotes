## Flyweight ##

### 内部状态（intrinsic properties）外部状态（extrinsic properties）的区别 ###

- intrinsic properties 指的是那些不随外界环境变化影响，可以唯一标识一个实例的属性。  
	换句话说，不同的 intrinsic 属性值将对应不同的实例。是判断缓存中是否已有该实例的标准。  
	形式上通常通过构造方法的参数传入值。
- extrinsic properties 指的是那些可由外界决定其值的属性，因此不能作为唯一标识一个实例的属性。  
	形式上，可以通过包括构造方法的各种设置方法传入值。也可以在实例创建后再传入值。
