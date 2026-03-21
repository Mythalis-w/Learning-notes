
```
// 强力滚动版本
let autoScroll = setInterval(() => {
    // 方法1：查找可能的滚动容器
    let scrollableElements = [
        document.documentElement,
        document.body,
        ...document.querySelectorAll('div, section, article, main')
    ].filter(el => 
        el.scrollHeight > el.clientHeight && 
        getComputedStyle(el).overflowY !== 'hidden'
    );
    
    // 找到最可能的滚动容器
    let target = scrollableElements[0] || document.documentElement;
    
    // 计算滚动距离（一屏的90%）
    let scrollAmount = target.clientHeight * 0.9;
    
    // 执行滚动
    target.scrollBy({
        top: scrollAmount,
        behavior: 'smooth'
    });
    
    console.log('滚动执行，目标：', target.tagName, '距离：', scrollAmount);
    
}, 5000);

console.log('自动滚动已启动，每5秒一次');
console.log('停止命令：clearInterval(autoScroll)');
```

在开发者工具中的console中输入以上代码。提前输入allow pasting 就可以了

结束只需要输入clearInterval(autoScroll)