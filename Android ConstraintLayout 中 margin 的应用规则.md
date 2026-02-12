# Android ConstraintLayout 中 margin 的应用规则
- 在 ConstraintLayout 中，margin 外边距必须依附于一个 Constraint 约束才能生效，margin 总是属于（并作用于）声明该约束的那个 View，也就是“拥有”约束的视图（而不是被约束到的目标视图）
- 比如 constraintStart_toEndOf 配合 marginStart、constraintTop_toBottomOf 配合 marginTop 等


##  谁“拥有”约束，margin 就作用在谁身上
- buttonA 的 marginStart="16dp" 作用于 buttonA 自己（应用于“拥有”该约束的视图），使它距离父容器左边缘 16dp
```xml
<Button
    android:id="@+id/buttonA"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    android:layout_marginStart="16dp" />
```


- buttonB 的 marginStart="8dp" 作用于 buttonB 自己，使它距离 buttonA 右边缘 8dp
- buttonB 的 start 约束到 buttonA 的 end（间距 8dp 是加在 buttonB 上的，即 buttonB 的左侧留出 8dp，而不是 buttonA 的右侧留出 8dp）
```xml
<Button
    android:id="@+id/buttonA"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    android:layout_marginStart="16dp" />
<Button
    android:id="@+id/buttonB"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toEndOf="@id/buttonA"
    android:layout_marginStart="8dp" />
```

## margin 设置在 buttonA 上行不行？
- buttonA 没有针对 buttonB 的约束（没有写 layout_constraintEnd_toStartOf="@id/buttonB"）
- 所以 marginEnd="8dp" 不会生效
```xml
<Button
    android:id="@+id/buttonA"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toStartOf="parent"
    android:layout_marginStart="16dp"
    android:layout_marginEnd="8dp"
     />
<Button
    android:id="@+id/buttonB"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:layout_constraintStart_toEndOf="@id/buttonA" />
```

谁写的constraintX_toYOf，marginX 
谁声明 margin，margin 就推开谁

## 双向约束


## goneMargin
- 目标视图隐藏时的 margin，同样作用于“拥有”约束的视图，仅当被约束的目标视图 visibility 为 GONE 时生效（当视图 A 变成 GONE 时，A 会缩小为一个点，导致视图 B 瞬间左移贴到 A 所在的位置）
- 通过设置视图 B 的 app:layout_goneMarginStart="50dp" 来解决

若用 layout_goneMargin...，也同样是“当前 View 在目标 gone 时使用的 margin”。
问题：。
解决：你可以
逻辑一致性：你看，即使针对“消失的对象”，控制权依然在“拥有约束”的视图 B 手里。
## 总结
- ConstraintLayout 中的 margin 必须配合约束（比如 layout_constraintTop_toTopOf）使用，否则不会生效
