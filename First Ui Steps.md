# How one line of code can wreck you program

## Context
In our game I was developing a main menu ui with the `kayak_ui` framework for the `Bevy Game Engine`  when I ran into this issue `QueryDoesNotMatch(3v0)`. 

I was confused by such a simple error, overthinking it's complexity. 

### Code Sample
Here is the code that panics
```rust
let props = props_query.get(entity).unwrap();
```
The problem was the `button_render` function was receiving the wrong entity from Bevy. So when I queried for it's props I was left with the mentioned error.

### My Problem Solving
My poor knowledge of the `kayak_ui` framework led me down a dark tunnel of doubt and fear of not wielding the proper skills to achieve something so simple as displaying a modular button.

I was frantically sifting through documentation I had already read for hours. This progressed over the span of days. 

After putting my code into an isolated environment I began to truly debug it. This is were we should talk about how to debug code

#### Go to Dev Debugging 
1. Always first step is to look for typos or anything thats missing like for example, importing a players health info in a `display_player_health` system would be pretty important. This was my first step in debugging, and spoiler it was my true problem.

2. If all your required data is being imported and there aren't any visible typos try adding `println!()`  statements. This will help analyze the data being imported. This was my second step, it didn't yield any valuable info to my problem. 

3. Try searching the internet, because 9 times out of 10 someone has done what you are trying do, this can also be something simpler like browsing documentation. For me this involved looking at other Bevy games that utilized the `kayak_ui` framework to try and determine how they could accomplish modular ui widgets. I also read documentation. This was my third and most demoralizing step because from what I could see in the docs, examples and other games, I was doing everything correct.

4. Modify your code to something else that could work. Think of how a scientific experiment has independent variables that can change, always change one at a time. For me this involved moving the button out of the main menu and instead just display the button itself. This yielded a program that worked but still didn't accomplish my overall goal of displaying the buttons within a confined menu space. Then I tried not rendering a button at all but rather just rendering the main menu. This led me to my real problem that I will talk about later.

5. Rewrite the whole thing. When I say the whole thing I don't mean the entire program, I mean rewrite the part thats broken. This should always be a final straw fix and should require a great amount of brainstorming on how to restructure, otherwise you’d just be doing the same thing over again. Luckily I didn't have to use this method here, but I have used it for previous projects and it has yielded great results.

### I found the bug
After changing my code in isolated ways, I narrowed down my issue to the `MainMenuBundle` which was how I was displaying the main menu. I examined it for a few minutes and couldn't find any obvious fallacies. I knew where the issue happened was in the `button_render` function. So I thought about using a default `kayak_ui` button instead to see if that would work. This led me to another clue, I got this error `Wasn't able to find render/update systems for widget: ButtonProps!`. This `ButtonProps` that it failed to find, was my own custom code, but I thought I removed all signs of it, and I did until I found the most elusive piece of code I didn't know about.
```rust
impl Default for MainMenuBundle {
    fn default() -> Self {
        Self {
            style: KStyle::default(),
            computed_styles: ComputedStyles::default(),
            children: KChildren::default(),
            on_event: OnEvent::default(),
            main_menu: MainMenu,
            widget_name: ButtonProps::default().get_name(),
        }
    }
}
```
Can you spot it? The `widget_name` parameter thats required for `kayak_ui` to identify which widget is which, is a `ButtonProps` name on the `MainMenuBundle`. This was essentially causing `kayak_ui` to give the wrong entity to the widget render function. Ultimately this error occurred because I copied some previous code for the `impl Default` and modified it slightly yet not completely for the `MainMenuBundle`. 

### What I learned
1. No matter what trust the process of debugging. Obviously every instance of debugging is unique and may require modifications to the original process. However in some way it branches off from the start.

2. Always check the code you copied to make sure any fallacies are corrected as this can become cumbersome to debug later as we’ve learned. 
