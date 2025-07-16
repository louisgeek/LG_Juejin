

##

```java
private void updateDimAmount(float dimAmount) {
    Dialog dialog = this.getDialog();
    if (dialog != null && dialog.getWindow() != null) {
        dialog.getWindow().setDimAmount(dimAmount); //0f 全透明 1f 全黑
    }
}

private void updateLayoutSize(int width, int height) {
    Dialog dialog = this.getDialog();
    if (dialog != null && dialog.getWindow() != null) {
        dialog.getWindow().setLayout(width, height); //任意一个为 0，可以实现事件穿透
    }
}
```