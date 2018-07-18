## 扁平化
数组的扁平化，就是将一个嵌套多层的数组 array (嵌套可以是任何层数)转换为只有一层的数组。

### 递归实现

    var arr = [1, [2, [3, 4]]];

    function flatten(arr) {
        var result = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            if (Array.isArray(arr[i])) {
                result = result.concat(flatten(arr[i]))
            }
            else {
                result.push(arr[i])
            }
        }
        return result;
    }
    
    
    console.log(flatten(arr))
    
### reduce实现

    var arr = [1, 2, [3, 4]]
    function flatten(arr) {
    	return arr.reduce(function(prev, next) {
    		return prev.concat(Array.isArray(next)? flatten(next) : next)
    	}, [])
    }
    
    
    console.log(flatten(arr))
    
### ES6扩展运算符实现
由于...[1, [2, [3, 4]]]得到的结果为[1, 2, [3, 4]],所以可以利用扩展运算符来遍历数组

    var arr = [1, 2, [3, 4]]
    var flatten = arr => {
    	while(arr.some(ele => Array.isArray(ele))) {
    		arr = [].concat(...arr)
    	}
    	return arr
    }
    
    console.log(flatten(arr))
    
### underscore实现

    var arr = [1, 2, [3, 4]]

    var newArr = _.flatten(arr)
    
    console.log(newArr)
    
## 去重

### ES6实现

    Array.form(new Set(arr))
    //或
    [...new Set(arr)]
    
### indexOf实现

    function unique(arr, isSort) {
    	var prev = []
    	var newArr = []
    	for(var i = 0; i < arr.length; i++) {
    		var value = arr[i]
    		if(isSort) {
    			if(!i || perv !== value) {
    				newArr.push(value)
    			}
    			prev = value
    		}
    		else if(newArr.indexOf(value) !== -1){
    			newArr.push(value)
    		}
    	}
    	return res
    }

## 最值

规则
- 如果有任一参数不能被转换为数值，则结果为 NaN。
- max 是 Math 的静态方法，所以应该像这样使用：Math.max()，而不是作为 Math 实例的方法 (简单的来说，就是不使用 new )
- 如果没有参数，则结果为 -Infinity (注意是负无穷大)


1.如果任一参数不能被转换为数值，这就意味着如果参数可以被转换成数字，就是可以进行比较的，比如：

    Math.max(true, 0) // 1
    Math.max(true, '2', null) // 2
    Math.max(1, undefined) // NaN
    Math.max(1, {}) // NaN
    
2.如果没有参数，则Math.max结果为 -Infinity，对应的，Math.min 函数，如果没有参数，则结果为 Infinity，所以：

    var min = Math.min();
    var max = Math.max();
    console.log(min > max);

### 遍历实现

    var result = arr[0];
    for (var i = 1; i < arr.length; i++) {
        result =  Math.max(result, arr[i]);
    }
    console.log(result);


### reduce实现

    function max(prev, next) {
        return Math.max(prev, next);
    }
    console.log(arr.reduce(max));   
    
### 排序实现

    arr.sort((a, b) => {
        a - b
    })
    console.log(arr[arr.length - 1])
    
### apply实现

    Math.max.apply(null, arr)
    
### ES6实现

    Math.max(...arr)
    
