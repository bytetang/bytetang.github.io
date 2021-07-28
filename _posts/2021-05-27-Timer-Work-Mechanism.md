---
title: "Android Timer Work Mechanism And Usage Guideline"
author: Morse
date: 2019-05-27
categories: [Blogging, Android]
tags: [Timer, Thread]
math: true
mermaid: true
---

# Background

Timer components are used for a time countdown in events and campaigns like rewards preview entries, tournament bucket, and tournament all-in-one list, invite friend events and so on. But it's dangerous if developers are not familiar with it. It's necessary to define the guideline for the usage of the timer. And this doc will list the cases we had faced in the past.


<b>How to do the countdown work?</b>

Use Timer scheduleAtFixedRate ,  execute tasks on `run()` of TimerTask class. multiple threads can share a single Timer object without the need for external synchronization.
Use CountDownTimer.  Handle the time on `onTick()` callback function. The callback is a synchronized block 

<b>The Lifecycle of Timer and CountDownTimer</b>

For Timer, after the last live reference to a Timer object goes away and all outstanding tasks have completed execution, the timer's task execution thread terminates gracefully (and becomes subject to garbage collection). However, this can take arbitrarily long to occur. Reference to 

[https://developer.android.com/reference/java/util/Timer](). 

So we know that Timer might not recycle automatically as we expected. 

For  CountDownTimer,  when calling the `start()` function.  The handler will send a message to the looper interval at the fixed time.  Execute on `onTicket()` when the `handleMessage()` . the onTicket is executed in a thread-safe way.   But the handler won’t stop until the `cancel()` call.

# Examples

## 1. Timer Leak in Activity / Fragment / Dialog

	
A task will be added into taskQuene when calling the sched or scheduleXX function.
  
The main looper thread consumes the task queue continuously. If the task finishes,  This task reference will be removed from the task queue. That’s why the task thread can be terminated peacefully. 



<b>When the leak occurs</b>

when the task is running, but the host page is destroyed.  Timer doesn’t know when the page has been destroyed,  so the task keeps running,  the task might hold the page reference, causing the page reference leak. Which we expect is that the task also needs to be terminal.

<b>How to resolve</b>

Calling the `cancel()` when the Activity `OnDestroy()`.  `Cancel()` function will clear the task queue.
All the references will be released,  and this is a thread-safe option. 

 
## 2. Timer used in RecyclerView

<b>ViewHolder lifecycle</b>

First we must know the recycling mechanism of RecyclerView. It's pretty different from the ListView.

When there is a new set of data to display on the screen, `onBindViewHolder` will be triggered to bind the data to the new item views for the user.But when these become invisible on screen (user scroll view away), the invisible views will be removed from the parent view(RecyclerView). Note the ViewHolder might be kept in memory for better performance,so at the time the variables have not been recycled yet.

If the user scrolls back to the previous item view, the `onBindViewHolder` will be triggered again. Developers need to rebind the ViewHolder data, but it might be obtained from the cached `ViewHolder`. 

Generally, When the ViewHolder is reused, resetting the ViewHolder fields will refresh the data and bind to the new views. So it’s always right whatever the user scrolls.

Then, what will happen when a timer is kept in the ViewHolder?

A ViewHolder with a CountDownTimer

<img width="521" alt="image" src="https://user-images.githubusercontent.com/6038077/127305083-99cf61c4-9158-4b7e-9c64-e0e52ddd7cb7.png">

The bindViewData function called by onBindViewHolder
 
```java
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.bindViewData("Timer$position")
}
```

Test scroll the list to bottom, then back to the top of the page.

Check the run result here: [https://drive.google.com/file/d/1G0iM1DtwV-4GVSk8BZqxFA35WjNjnhi6/view?usp=sharing]()


You can find the count-down text performs not as expected and flash continuously, Seems another timer still updates the same item view. onBindViewHolder event reset the Timer when the RecyclerView starts to reuse, so the countDownTimer value be covered with a new instance,  but what causes the text flash?

<b>Root Reason</b>


As the "ViewHolder Behavior" chapter said, the ViewHolder might be cached and reused, then the timer still kept in memory, through the countDownTimer be reset to a new instance,  but the timer differs with other components.  CountDownTimer maintain a thread looper and interval send the message to the onTicket() , the lifecycle of looper longer than ViewHolder, if ViewHolder need to recycled or reset, the memory can’t released since the anonymous CountDownTimer caused the leak happened, so there are multiple CountTimer exists. That’s why the text flashes.


<b>How to resolve</b>

Manually cancel the timer when the ViewHolder recycle or detach

<img width="513" alt="image" src="https://user-images.githubusercontent.com/6038077/127305416-04b6d02a-903a-4e41-be64-87112e2278bd.png">

If you want to countdown from the last time, save the last time on OnViewDetachedFromWindow() and recover the last time countdown on onViewAttachedToWindow is a good choice.

# Summaries

The core of the approach to resolve the Timer leak is simple:
<b>Call Timer cancel at the right time!!!</b>

- For Activity, cancel the Timer in `OnDestory()` , for Fragment cancel in `onDestroyView()`
- For RecyclerView or GridView, cancel and release Timer in     `onViewDetachedFromWindow()` or `onViewRecycled()`
