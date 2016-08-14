% Kotlin + Data binding での List adapter
% 2016-8-14

# Kotlin + Data binding での List adapter

Kotlin + Data binding を用いた場合の List adapter 実装. 個人的ベストプラクティス.  
これさえ作っておけば標準的な List に対応できる.

## adapter

```kotlin
class BindingListAdapter<T>(private val context: Context, private val layout: Int, private val items: List<T> = emptyList<T>()) : BaseAdapter() {

    override fun getItem(position: Int): T {
        return items[position]
    }

    override fun getItemId(position: Int): Long {
        return position.toLong()
    }

    override fun getCount(): Int {
        return items.size
    }

    override fun getView(position: Int, convertView: View?, parent: ViewGroup?): View {
        val view: View
        val binding: ViewDataBinding
        if (convertView == null) {
            binding = DataBindingUtil.inflate(LayoutInflater.from(context), layout, parent, false)
            view = binding.root
        } else {
            view = convertView
            binding = DataBindingUtil.getBinding(view)
        }

        binding.setVariable(BR.vm, getItem(position))

        return view
    }

}
```

constructor には context, layout resource id, item を渡す.  
T は List item ViewModel. getView で ViewDataBinding と紐づける.  

## ViewModel

```kotlin
class DemoViewModel(context: Context) {
    val adapter = BindingListAdapter<DemoListItemViewModel>(context, R.layout.list_item_demo)
}
```

### List item ViewModel

```kotlin
class DemoListItemViewModel {

    val title = "title"

    val description = "description"

}
```

view model は特に工夫せず素直に作るだけ.  
値が可変で通知が必要であれば, もちろん以下のようにする必要がある.

```kotlin
class DemoListItemViewModel : BaseObservable() {

    @Bindable
    var title = "title"
        private set

    val description = "description"

    fun update() {
        title = "hi!"
        notifyPropertyChanged(BR.title)
    }

}
```

## Layout xml

List view に adapter を紐付ける.  
List item layout xml では, 内部の要素をそれぞれ紐づけることで対応する.

```xml
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable
            name="vm"
            type="jp.cordea.demo.viewmodel.DemoViewModel" />
    </data>

    <ListView
        app:adapter="@{vm.adapter}"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</layout>
```

### List item

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable
            name="vm"
            type="jp.cordea.demo.viewmodel.DemoListItemViewModel" />
    </data>

    <LinearLayout
        android:orientation="vertical"
        android:gravity="center_vertical"
        android:layout_height="72dp"
        android:layout_width="match_parent">

        <TextView
            android:text="@{vm.title}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

        <TextView
            android:text="@{vm.description}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

    </LinearLayout>
</layout>
```

