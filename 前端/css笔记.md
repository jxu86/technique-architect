
### 四个div在父div居中排列
```html
<div class="parent">
  <div class="child"></div>
  <div class="child"></div>
  <div class="child"></div>
  <div class="child"></div>
</div>
```

```css
.parent {
  display: flex;
  justify-content: center;
}

.child {
  width: 50px;
  height: 50px;
  background-color: #ccc;
  margin: 0 10px;
}
```