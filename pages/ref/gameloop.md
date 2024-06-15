---
layout: page
title: Game Loop
permalink: /ref/gameloop
categories: ref game_programming
---

## References

- gameprogrammingpatterns : [game loop][]
- gafferongames : [fix your timestep][]

## Most Basic Loop

The game loop is at the heart of a game application and determines how fast the
application runs. The most basic loop is similar to the following:

```c++
while (is_running) {
  process_input();
  update();
  render();
}
```

This loop simply runs as fast as it can with no way to control the speed, which 
will vary from machine to machine.

## Sleeping

The loop frequency can be limited with a simple timer:

```c++
// 60 FPS ~ 16ms/frame
double max_loop_duration = 60.0/1000.0; 

double start_time = 0.0;
double end_time = 0.0;
double loop_duration = 0.0;
double duration_remaining = 0.0;

while (is_running) {
  start_time = get_current_time();
  
  process_input();
  update();
  render();
  
  end_time = get_current_time();
  loop_duration = end_time - start_time;
  duration_remaining = max_loop_duration - loop_duration;
  
  sleep(duration_remaining);
}
```

However, a problem arises if the loop duration is ever longer than the desired
maximum; The process cannot sleep for a negative duration.

## Variable Timestep

A timestep can be passed to the `update` function, scaling all updates. 
If the loop takes longer to run, a larger step will be taken during the update;
This ensures that the output is consistent, whether it takes 2 updates or 200.

```c++
double current_time = 0.0;
double delta_time = 0.0;
double last_time = get_current_time();

while (is_running) {
  current_time = get_current_time();
  delta_time = current_time - last_time;
  last_time = current_time;
  
  process_input();
  update(delta_time);
  render();
}
```

This works fine in the abstract, but, computers are real and can only offer an
approximation of the abstract. Due to floating-point rounding errors, etc. this
game loop is no longer deterministic; Output will differ depending on the time 
step which will likely cause de-syncing within a networked game.

<aside>A small quirk of this method is that the current frame is actually 
updated according to the duration of the _previous frame_.</aside>

## Fixed Timestep with Decoupled Rendering

The previous two methods can be somewhat combined. The game loop adds time to 
an accumulator, which is then "consumed" by the `update` function in fixed
steps.

The `update` function can be called from a sub-loop, allowing it to execute at 
a higher frequency than the `render` (which is often capped due to v-sync). A
shorter timestep for the update function is preferred as it offers better 
stability within the physics simulation.

The main `update` function can also be split up to allow more "expensive" code 
to run at a slower pace. For example; If the AI calculations took 40ms, 
a unified update function would limit game to a maximum of 25 updates/second. 

This code does not necessarily need to run that fast, so, by splitting it into
its own function and running it at a lower frequency (e.g. 10 updates/second)
the amount of work done by the main loop is reduced, allowing for a higher 
update rate (i.e. a lower timestep) for the physics simulation.

```c++
struct LoopTimer {
  double current_time = 0.0;
  double last_time = 0.0;
  double delta_time = 0.0;
}

struct InnerLoop {
  double accumulator = 0.0;
  double timestep = 1.0/1000.0;
  int iterations_max = 0;
  int iteration_count = 0;
};

InnerLoop update_loop;
update_loop.timestep = 200.0/1000.0;
update_loop.iterations_max = 3;

InnerLoop ai_loop;
ai_loop.timestep = 10.0/1000.0;]

InnerLoop draw_loop;
draw_loop.timestep = 60.0/1000.0;

LoopTimer loop_timer;
loop_timer.last_time = get_current_time();

while (is_running) {
  loop_timer.current_time = get_current_time();
  loop_timer.delta_time = loop_timer.current_time - loop_timer.last_time;
  loop_timer.last_time = loop_timer.current_time;
  
  update_loop.accumulator += loop_timer.delta_time;
  ai_loop.accumulator += loop_timer.delta_time;
  
  process_input();
  
  update_loop.iteration_count = 0;
  while (update_loop.accumulator >= update_loop.timestep) {
    update(update_loop.timestep);
    update_loop.accumulator -= update_loop.timestep;
    
    ++update_loop.iteration_count;
    if (update_loop.iteration_count >= update_loop.iterations_max) {
      break;
    }
  }

  if (ai_loop.accumulator >= ai_loop.timestep) {
    update_ai();
    ai_loop.accumulator -= ai_loop.timestep;
  }

  if (draw_loop.accumulator >= draw_loop.timestep) {
    render();
    draw_loop.accumulator -= draw_loop.timestep;
  }
}
```

The problem of the update loop taking too long still remains. In addition, a 
"lag spike" can cause a lock up of the loop. This is "dealt with" by limiting 
the number of updates permitted per loop, allowing a frame to be rendered, and
hoping that the simulation will catch up eventually. 

The update loop will still be behind and need to catch up, but intermittent 
rendering may lessen the perceived slow down.

### A note on decoupled rendering

As the updating and rendering are decoupled, it is possible (and likely) that a
frame will be rendered in between two updates.

This can be addressed by a more advanced rendering engine capable of 
interpolation or extrapolation. (and, perhaps, further division of the physics
update into "positional updates" and "collision detection and correction").

#### Interpolation

The game keeps track of the previous game state and the renderer interpolates
between the two (using `update_loop.accumulator`). This adds one timestep of 
latency to the game.

#### Extrapolation

The renderer extrapolates object positions using their current velocity (using
`update_loop.accumulator`). The predicted outcome may be erroneous, potentially
causing visual artefacts/stuttering as the game state is corrected in the next 
frame.

[game loop]: <https://gameprogrammingpatterns.com/game-loop.html>
[fix your timestep]: <https://gafferongames.com/post/fix_your_timestep/>
