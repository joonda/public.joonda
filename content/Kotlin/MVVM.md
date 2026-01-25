---
title: MVVM Pattern
description: MVVM-pattern
tags:
  - Kotlin
  - Android
date: 2026-01-25
draft: false
---
# MVVM
- 애플리케이션을 데이터를 처리하는 모델 (Model), 사용자에게 보여지는 UI인 뷰 (View), 뷰에 바인딩 되어 모델과 뷰 사이를 이어주는 뷰 모델 (View Model)로 분리하는 방식
- 모델과 뷰 뿐만 아니라 뷰와 뷰 모델간의 의존성까지 최소화한 형태로, UI가 실제 코드와 거의 완벽하게 분리되어 있다.

## Model
- 도메인 중심 
- 핵심 데이터와 규칙을 가진다.
- Entity / data class, Domain logic, Repository interface 등이 포함된다
- UI는 절대 알지 못하며, 어디든 사용해도 동일해야한다.

```kotlin title="CounterModel.kt"
data class CounterModel(var count : Int)  
  
class CounterRepository {  
    private var _counter = CounterModel(0)  
  
    fun getCounter() = _counter  
  
    fun incrementCounter() {  
        _counter.count++  
    }  
  
    fun decrementCounter() {  
        _counter.count--  
    }  
}
```

## ViewModel
- Model을 View가 이해할 수 있는 형태로 가공한다.
- 화면 상태 관리, 사용자 이벤트 처리, 비즈니스 로직 호출 등
- View에 대한 참조가 없으며 상태 중심이다.

```kotlin title="CounterViewModel.kt"
class CounterViewModel() : ViewModel() {  
    private val _repository: CounterRepository = CounterRepository()  
    private val _count = mutableStateOf(_repository.getCounter().count)  
  
    val count: MutableState<Int> = _count  
  
    fun increment() {  
        _repository.incrementCounter()  
        _count.value = _repository.getCounter().count  
    }  
  
    fun decrement() {  
        _repository.decrementCounter()  
        _count.value = _repository.getCounter().count  
    }  
}
```

## View
- UI만 담당
- 사용자 입력을 ViewModel에 전달
- ViewModel의 상태를 관찰하고 그린다.

```kotlin title="MainActivity.kt"
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
        enableEdgeToEdge()  
        setContent {  
            val viewModel: CounterViewModel = viewModel()  
            CounterMVVMTheme {  
                Scaffold(modifier = Modifier.fillMaxSize()) { innerPadding ->  
                    TheCounterApp(  
                        modifier = Modifier.padding(innerPadding),  
                        viewModel  
                    )  
                }  
            }        
        }    
	}  
}  
  
@Composable  
fun TheCounterApp(modifier: Modifier = Modifier, viewModel: CounterViewModel) {  
  
    Column(modifier = Modifier.fillMaxSize(),  
        verticalArrangement = Arrangement.Center,  
        horizontalAlignment = Alignment.CenterHorizontally) {  
        Text(text = "Count: ${viewModel.count.value}",  
            fontSize = 24.sp,  
            fontWeight = FontWeight.Bold  
        )  
        Spacer(modifier = Modifier.height(16.dp))  
        Row{  
            Button(onClick = {viewModel.increment()}) {  
                Text("Increment")  
            }  
            Spacer(modifier = Modifier.width(8.dp))  
            Button(onClick = {viewModel.decrement()}) {  
                Text("Decrement")  
            }  
        }    
	}
}
```

# Reference

- [MVVM - 나무위키](https://namu.wiki/w/MVVM)
- [Udemy Lecture](https://www.udemy.com/course/best-android-12-kotlin/)