# randomColor
JavaScript随机生成颜色标签，并且避免出现相近的颜色以及黑色、白色、灰色

实际需求是生成随机颜色的标签，但是要求颜色较为鲜艳，不要出现黑色灰色 以及太接近背景颜色的白色。

我刚用vue写了一个demo：

```
<div id="app">
  <span class="box" v-for="item in 16" :style="randomRgb()"></span>
</div>
```

```
new Vue({
  el: '#app',
  methods: {
    randomRgb: function () {
      var R = Math.floor(Math.random() * 255);
      var G = Math.floor(Math.random() * 255);
      var B = Math.floor(Math.random() * 255);
      return { background: 'rgb(' + R + ',' + G + ',' + B + ')' };
    }
  }
});
```

甚至能生成了这样的接近色：

![图片描述][1]

避免灰色，黑白色其实我已经有思路了，但避免接近色方面不是很有想法，请教一下各位。

----------

感谢各位提供的思路，我把demo进行了完善，代码如下：

```
<div id="app">
  <span class="box" v-for="item in rgbArray" :style="item.style"></span>
</div>
```

```
new Vue({
  el: '#app',
  data: {
    hslArray: []
  },

  created: function () {
    this.hslArray = this.getHslArray();
  },

  computed: {
    rgbArray: function () {
      var self = this;
      if (!self.hslArray.length) return [];

      var rgbArray = self.hslArray.map(function (item) {
        return self.hslToRgb.apply(this, item);
      });

      return rgbArray.map(function (item) {
        return {
          value: item,
          style: { background: 'rgb(' + item.toString() + ')' }
        }
      });
    }
  },

  methods: {
    /**
     * HSL颜色值转换为RGB
     * H，S，L 设定在 [0, 1] 之间
     * R，G，B 返回在 [0, 255] 之间
     *
     * @param H 色相
     * @param S 饱和度
     * @param L 亮度
     * @returns Array RGB色值
     */
    hslToRgb: function (H, S, L) {
      var R, G, B;
      if (+S === 0) {
        R = G = B = L; // 饱和度为0 为灰色
      } else {
        var hue2Rgb = function (p, q, t) {
          if (t < 0) t += 1;
          if (t > 1) t -= 1;
          if (t < 1/6) return p + (q - p) * 6 * t;
          if (t < 1/2) return q;
          if (t < 2/3) return p + (q - p) * (2/3 - t) * 6;
          return p;
        };
        var Q = L < 0.5 ? L * (1 + S) : L + S - L * S;
        var P = 2 * L - Q;
        R = hue2Rgb(P, Q, H + 1/3);
        G = hue2Rgb(P, Q, H);
        B = hue2Rgb(P, Q, H - 1/3);
      }
      return [Math.round(R * 255), Math.round(G * 255), Math.round(B * 255)];
    },

    // 获取随机HSL
    randomHsl: function () {
      var H = Math.random();
      var S = Math.random();
      var L = Math.random();
      return [H, S, L];
    },

    // 获取HSL数组
    getHslArray: function () {
      var HSL = [];
      var hslLength = 16; // 获取数量
      for (var i = 0; i < hslLength; i++) {
        var ret = this.randomHsl();

        // 颜色相邻颜色差异须大于 0.25
        if (i > 0 && Math.abs(ret[0] - HSL[i - 1][0]) < 0.25) {
          i--;
          continue; // 重新获取随机色
        }
        ret[1] = 0.7 + (ret[1] * 0.2); // [0.7 - 0.9] 排除过灰颜色
        ret[2] = 0.4 + (ret[2] * 0.4); // [0.4 - 0.8] 排除过亮过暗色

        // 数据转化到小数点后两位
        ret = ret.map(function (item) {
          return parseFloat(item.toFixed(2));
        });

        HSL.push(ret);
      }
      return HSL;
    }
  }
});
```
避免了黑白灰色，以及相邻近色，demo效果如图：

![图片描述][2]


  [1]: https://sfault-image.b0.upaiyun.com/846/982/846982242-599cfa9711998
  [2]: https://sfault-image.b0.upaiyun.com/393/961/3939612001-599e3eebbf275
