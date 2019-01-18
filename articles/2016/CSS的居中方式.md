网页开发过程中，经常会遇到dom元素居中的情况，即让某些元素相对父元素居中。不同情况有不同处理的方式，比如父元素的宽高是固定的与非固定的，其处理方式会有差别。下面是几种常见的方式。
1. **水平方向居中**
  * 内联元素：为其父级元素设置属性`text-align: center;`
  * 块级元素：
    * 为居中元素设置属性`margin: 0 auto；`
    * 使用flex布局（如果是webkit内核的浏览器，需要加上前缀`－webkit-`）
    ```css
    .parent {
      display: flex;
      justify-content: center;
    }
    ```
2. **垂直方向居中**
  * 内联元素（单行）
    * 为其父级元素设置相等的`padding-top`与`padding-bottom`，且需满足`padding-top + padding-bottom + font-size = parent-height`
    * 为其父级元素设置`line-height`属性，其值等于`height`
  * 内联元素（多行）
    * 设置父级display属性为`table`，居中元素的display属性设置为`table-cell`，并添加属性`vertical-align: middle;`
      ```css
      #parent {
        display: table;
        height: 300px;
        width: 200px;
        background: #eee;
      }
      #child {
        display: table-cell;
        vertical-align: middle;
      }
      ```
  * 块级元素
    * 已知居中元素高度并且固定不变
      ```css
      .parent {
        position: relative;
      }
      .child {
        position: absolute;
        top: 50%;
        height: 100px;
        margin-top: -50px; /* height的一半大小 */
      }
      ```
    * 不知居中元素的高度
      ```css
      .parent {
        position: relative;
      }
      .child {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
      }         
      ```
    * 使用flex布局
      ```css
      .parent {
        display: flex;
        flex-direction: column;
        justify-content: center;
      }
      ```
3. **水平方向与垂直方向同时居中**
  * 已知居中元素的宽高，并且固定不变
    ```css
    .parent {
      position: relative;
    }

    .child {
      width: 300px;
      height: 100px;
      padding: 20px;

      position: absolute;
      top: 50%;
      left: 50%;

      margin: -50px 0 0 -150px;
    }
    ```
  * 不知居中元素的宽高
    ```css
    .parent {
      position: relative;
    }
    .child {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
    ```
  * 使用flex布局
    ```css
    .parent {
      display: flex;
      justify-content: center;
      align-items: center;
    }
    ```
  * 使用calc计算
    ```html
    <div id="parent">
        <div id="child"></div>
    </div>
    ```

    ```css
    #parent {
      width: 300px;
      height: 200px;
      border: 1px solid;
      position: relative;
    }
    #child {
      width: 40%;
      height: 40%;
      background: green;
      position: absolute;
      top: calc(50% - (40% / 2));
      left: calc(50% - (40% / 2));
    }
    ```
