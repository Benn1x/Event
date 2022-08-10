# Example Code:
```rust
use event::*;
use std::sync::atomic::Ordering;
use tokio::time::{sleep, Duration};
use std::sync::Arc;
use std::sync::atomic::AtomicU32;
#[tokio::main(flavor = "multi_thread", worker_threads = 8)]
async fn main() {

    let handler_count1 = Arc::new(AtomicU32::new(0));
    let hc1 = handler_count1.clone();

    let handler_count2 = Arc::new(AtomicU32::new(0));
    let hc2 = handler_count2.clone();

    let handler_count2 = Arc::new(AtomicU32::new(0));
    let hc2 = handler_count2.clone();

    let handler_count3 = Arc::new(AtomicU32::new(0));
    let hc3 = handler_count3.clone();

    #[derive(Clone)]
    struct TestEvent(i32, i32);
    let dispatcher = Dispatcher::<TestEvent>::new();

    dispatcher.add(move |events| -> bool {
        hc1.fetch_add(1, Ordering::SeqCst);
        println!("[Event Handler #1] TestEvent({})", events.0);
        true
    }); 

    dispatcher.add(move |event| -> bool {
        hc2.fetch_add(1, Ordering::SeqCst);
        println!("[Event Handler #2] TestEvent({})", event.0);

        event.0 != 8
    }); 
    dispatcher.add(move |a| -> bool{
        hc3.fetch_add(1,Ordering::SeqCst);
        println!("[Event Handler #3] TestEvent.2({})", a.1);
        true
    }); 
    sleep(Duration::from_millis(50)).await;

    for i in 0..10 {
        dispatcher.dispatch(TestEvent(i + 1,1011));
	}
    sleep(Duration::from_millis(1)).await;
}
```

# What does What
#### `let dispatcher = Dispatcher::<TestEvent>::new();`
This Creates the Event for the `TestEvent` Struct

#### `dispatcher.add(move |events| -> bool {});`
This lamnda gets Executet if the Event gets Called
events is the for the Event
in This Case is events just an synonyme for the struct `TestEvent`

#### `dispatcher.dispatch(TestEvent());`
This triggers the Event and executes the lamndas out of `dispatcher.add();`
