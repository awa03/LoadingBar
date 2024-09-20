# Timer Example

This timer example demonstrates how to build a basic application using the loadingBar library.

## Namespace & Preprocessor Statments
```cpp
#include "../include/Animation.hpp"
#include "../include/Bar.hpp"
#include "../include/Symbols.hpp"
#include <chrono>
#include <thread>

using namespace std;
using namespace loadingBar;

```

We will include the `Animation`, `Bar`, and `Symbols` file in order to create a cohesive timer app. The Animation file will provide us with a way to create animations for our leading character. The leading character is the `>` within this sample loading bar `--->`. Additionally the Animation file contains some premade animations that you may use within your projects. These can be accessed through the `premade` namespace, which is encapsulated within the animation class. An example of this maybe seen as `premade::loading`; 

> To View More Info on the Animation header please visit the [Animation Documentation]()

The Bar file will allow us to create the actual loading bar. This is the only nessicary file within this list, however I will recommend including the Animation and Symbols file. 

> To View More Info on the Bar header please visit the [Bar Documentation]()

As for the symbols file, this file only contains characters which maybe useful in creating animations. 

> To View More Info on the Symbols header please visit the [Symbols Documentation]()


## Command Line Arguments

```cpp
int main(int argc, char* argv[]) {
    if (argc < 2) {
        cerr << "Usage: timer {seconds}" << endl;
        return 1;
    }

    int seconds = atoi(argv[1]);
    if (seconds <= 0) {
        cerr << "Please provide a positive number of seconds." << endl;
        return 1;
    }
```

When creating a timer this section is completely optionaly, but I would encourage the creation of a dynamic timer even though it may make animations more difficult to implement. I may in the future implement a subclass in order to abstract some of the math away from the user. I will then cast this to a int in order to perform our calculations in order to find how often to change frames, as well as how often to refresh the bar. 

## Creating a Bar

The first thing we need to do is initialize a bar variable. This will be the driving force behind our program. Then we will set the width, this is how long the bar will be. Then we will initialize the fill, remain, and animations. The set fill will set the character used for the remaining spaces. If you would like a clear remaining just use " ". The none animation represents this, for animations not symbols.

```cpp
    loadingBar::bar b;
    b.set_width(30);
    b.set_fill(symbol::bar);
    b.set_remain(symbol::fade_bar);
    b.set_lead_animation(premade::none);
```

## Calculations

There are a few benifits to low level control. First of all, this program in some way must be user controlled in order to allow program integration. This comes with its pros as well as its cons, but it wont be too tricky I promise! 

First we need to convert to milliseconds. This is useful because we wont lose too much precision from our later division, and through float casting. We will then initialize the increments count to the width of the bar, this will allow us to evenly space our timer. Then we will divide the total time by the number of increments in order to find out how much time each increment should take. 

Take for example a 10 second timer. If we were to have a timer bar length of 10, the `total_time = 10 * 1000`. This will resolve to the total time being `10000`. The `time_per_increment` will then be `10000 / 10`, this will then resolve to `1000`.

```cpp
    int total_time = seconds * 1000; 
    int increments = b.get_width(); 
    int time_per_increment = total_time / increments;
```

# The Timer Loop

The timer loop incorporates the `time_per_increment` within these in order to find how long each bar should be. We then run this loop until the clock has reached the increment count. Meaning there would be `1000` ms per increment, as shown in the aforementioned example.

```cpp
    // Main loop until the bar is complete
    for (int i = 0; i < increments; i++) {
        auto start = std::chrono::steady_clock::now();

        b.flush();
        b.print_bar();
        b.add_progress();
        b.next_lead_frame();

        auto end = std::chrono::steady_clock::now();
        auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count();

        // Sleep for the remaining time
        std::this_thread::sleep_for(std::chrono::milliseconds(time_per_increment - elapsed));
    }

    return 0;
}
```
