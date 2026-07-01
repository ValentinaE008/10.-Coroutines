Домашнее задание к занятию «10. Coroutines: Scopes, Cancellation, Supervision»
Выполненное задание прикрепите ссылкой на ваши GitHub-проекты в личном кабинете студента на сайте netology.ru.

Вопросы: Cancellation
Вопрос №1
Отработает ли в этом коде строка <--? Поясните, почему да или нет.

fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
    }
    delay(100)
    job.cancelAndJoin()
}
Ответ:
Нет, строка не отработает. Причина в том, что код запускает две корутины, которые обе задерживаются на 500 миллисекунд. Однако после задержки в 100 миллисекунд, вызывается job.cancelAndJoin(), что отменяет все корутины, запущенные внутри этого job. Этот вызов происходит до того, как ни одна из корутин успевает завершить свою работу, поэтому строка, содержащая println("ok"), не будет достигнута.

Вопрос №2
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() = runBlocking {
    val job = CoroutineScope(EmptyCoroutineContext).launch {
        val child = launch {
            delay(500)
            println("ok") // <--
        }
        launch {
            delay(500)
            println("ok")
        }
        delay(100)
        child.cancel()
    }
    delay(100)
    job.join()
}
Ответ:
Нет, строка не отработает. Все аналогично первому вопросу, но с дополнительной деталью. Здесь корутина child.cancel() вызывается после задержки 100 миллисекунд, что приводит к отмене только корутины child. Однако в этом коде есть другая корутина, которая также запускается, но так как child будет отменена, строка // <-- в ней не будет выполнена.

Вопросы: Exception Handling
Вопрос №1
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    with(CoroutineScope(EmptyCoroutineContext)) {
        try {
            launch {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
Ответ:
Нет, строка не отработает. Когда исключение выбрасывается в корутине, оно не перехватывается в try-catch, так как launch возвращает Job, и управление не возвращается в блок try. Исключение будет выброшено и не будет обработано в catch, поэтому e.printStackTrace() не будет вызван.

Вопрос №2
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
Ответ:
Нет, строка не отработает. Исключение выбрасывается внутри корутины, находящейся под управлением coroutineScope, что приводит к тому, что оно не может быть перехвачено в try-catch, содержащемся в родительской корутине. Обработка исключения произойдет вне launch, и catch не будет выполнен.

Вопрос №3
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                throw Exception("something bad happened")
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
Ответ:
Нет, строка не отработает. Подобно предыдущим вопросам, исключение возникает внутри корутины, управляемой supervisorScope, и не будет перехвачено в родительском try-catch, поскольку обработка происходит в другом контексте.

Вопрос №4
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            coroutineScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    Thread.sleep(1000)
}
Ответ:
Да, строка отработает. В этом случае вы запустили корутины внутри coroutineScope. Когда одна из корутин выбрасывает исключение, это приводит к отмене всей корутины с coroutineScope. Исключение будет перехвачено в блоке catch.

Вопрос №5
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        try {
            supervisorScope {
                launch {
                    delay(500)
                    throw Exception("something bad happened") // <--
                }
                launch {
                    throw Exception("something bad happened")
                }
            }
        } catch (e: Exception) {
            e.printStackTrace() // <--
        }
    }
    Thread.sleep(1000)
}
Ответ:
Нет, строка не отработает. Аналогично вопросу №4, исключение будет выброшено в supervisorScope, но так как supervisorScope не отменяет другие корутины при возникновении исключения, оно не будет перехвачено в блоке catch родительской корутины, и e.printStackTrace() не выполнится.

Вопрос №6
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
Ответ:
Нет, строка не отработает. Исключение выбрасывается в корутине, и поскольку оно происходит до завершения всех запущенных корутин, это не даст возможности выполнить println("ok").

Вопрос №7
Отработает ли в этом коде строка <--. Поясните, почему да или нет.

fun main() {
    CoroutineScope(EmptyCoroutineContext).launch {
        CoroutineScope(EmptyCoroutineContext + SupervisorJob()).launch {
            launch {
                delay(1000)
                println("ok") // <--
            }
            launch {
                delay(500)
                println("ok")
            }
            throw Exception("something bad happened")
        }
    }
    Thread.sleep(1000)
}
Ответ:
Да, строка отработает. Использование SupervisorJob позволяет другим корутинам продолжать выполнение, даже если одна из них бросает исключение. Поэтому в данном случае обе корутины будут выполнены до момента выброса исключения.

Итоги
Вопросы 1, 2, 3, 5, 6 не отработают строку // <--.
Вопросы 4 и 7 — строка отработает.