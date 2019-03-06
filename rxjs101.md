# RxJS 101.1 Magic Faucet, Fruit Counting and Why Everything Could Be Stream

Thanks to @bdong-404 finished the first half of this blog, and @Khuang-va provided some examples.

## Intro

There are four quadrants to describe the knowledge, "we know we know", "we don't know we know", "we know we don't know" and "we don't know we don't know".

of course, `:jrocks: Mr. Rans loves Rock`  should be in "we know that we know" quadrant, and `how to master c++ in 21 days` belongs to "we know we don't know" quadrant.

but how about RxJS? 

well, I will vote for `we don't know we don't know` quadrant. it is easy to compile and make it works, but sometimes really hard to make them work correctly. and the most dangerous part is, sometimes we don't know it does not work correctly.

so I write this.
## What is RxJS
before that, lets talk about `Reactive`.
and since copy paste definition would not help at all, let's start with a joke.

> everyone know that last VendastaCon is a success. but there is one thing not everyone know: we have a participant who came from a far away desert tribe, he was coming here for a solution for his social media and reputation management like others. but when he arrived Banff, one thing draw his attention: faucet. this is the first time he have seen such a beautiful thing. just turn it on, water will come out. finding water is such a hard thing in his tribe! comparing with faucet, even our marketplace or reputation management is not that important anymore.

> when VendastaCon ends, he did not become our partner, but went home with 20 faucets.

a funny one, right? 
now let's close eyes and image: what if it is not a joke?

what if we get the technology, magic faucets, you can bring it to anywhere, give it to anyone, and when you turn it on, water always come out. it not even need to be water! wine, beer, vodka... wait, it not even need to be liquid! hotdogs, coins, books... 

of course, it is not come from no where (that will break law of conservation of mass!), we have a ultra dimension pipe system behind it. we can transfer matter freely from one place to another.

how would this change our life?

human may not need to transfer or exchange matters anymore. all we need to give or get are faucets. water bottle or bucket? nah, just bring the faucet, and turn it on when you really need the water. 

and you know what, another great benefit is, people do not make sandwich anymore. they design the ultra dimension pipe like `take bread, cheese and tomato slice as input, then give me sandwich`.

sounds neat, huh?

then I guess you already understand the basic part of `Reactive Programming` now.
`RxJS` just another way of saying `implement ReactiveX in Javascript`.

### Everything is stream

just like we talked in previous chapter. since we have the stream of `water`, why not stream of `integer`, `boolean` or `object`?
image that, we have a faucet, it has a tag on it says `isElasticSearchDown()`, everytime you turn it on, it will start the stream of `boolean`. every time elastic down, this faucet will give you a `true`, and it will magically become `false` after elastic search back to work.  we will never need to pass the boolean itself to anywhere, all we need is just this faucet. or use professional way to say, `Observable<Boolean>`.

I know you are thinking `but, why?`, we will answer this in next chapter.

### Manipulate data states, not data itself
in the world with such magic faucet, carrying water manually will make you looks silly. in any case we need to `transfer` water, we just need to transfer the faucet. we only need to get water when we really need to `consume` water. 
in our professional word, the Observable almost **never** need to be unpacked, until we really need it, like rendering in browser, or mapping the data from one form to another. but in second case, we almost always pack it back to Observable.

Yeah I hear you, `BUT, WHY?`. clear and loud.
let talk about this.


## Why RxJS?
before we talk about this, let us recall one thing: **do you remember how to handle memory leak in Java?**

you don't know? good.
because the answer is, most of the time, we do not care, because we do not need to.

Same thing happens in RxJS. if you follow the principle of RxJS, then you do not need to worry **async** in most of time.

here is an example.
in non-RxJS world, which is regular programmers' world, a simple calculating will looks like this:

```javascript
let apple = 1;
let pear = 2;
let fruit = apple + pear;
let apple = 3;
let pear = 5;
```

if we try to get the result of `fruit`, we are calculating `1+2`, the line `fruit = apple + pear` is actually saying
>  "hey, get me the value of apple and pear, add them together, and assign the value to fruit."

when we calling that line, it is an *instant* order, it start the calculating, it will get the *instant* value of `apple` and `pear`, use it, discard it, and would not care if this value changed or not after the calculation done, just like a bad relationship starts in Vagas. 
this is what we say `Manipulate Data`. if we going with magic faucet world, it is like `carrying water with you`.

Things goes very different in RxJS world. 
```javascript
apple$$.next(1);
pear$$$.next(2);
fruit$.pipe(
  combineLatest(apple$$, pear$$),
  map((apple, pear) => {
    return apple + pear;
  })
);
apple$$.next(3);
pear$$$.next(5);

```

when we trying to get (called `subscribe` in RxJS) fruit after everything, it will be 8 instead of 3. it is because in the fruit pipe we are not giving an **order**, but a **definition**. we are actually saying:
> Hey `fruit`, since today, your value will always be the sum of `apple` and `pear`, anytime when they change their values, you should change yours,for better, for worse, for richer, for poorer, in sickness and in health, to love and to cherish, till `unsubscribe` do us part

we do not `manipulate data` anymore, we start to define or describe `how data should be processed`. 

this is like the faucet world, we are not carrying the water bucket anymore, but `define the pipe behind the faucet`, and only carry the faucet with us. I do not need to  call method `hey would you mind to calculate apple and pear for me again?` the faucet  always have newest data. Actually in RxJS, we also have all history of that data by using `ReplaySubject`


### Example of how RxJS benefit us

here is an example in our project:
![image](https://storage.googleapis.com/pguo/rxjs-stream-example.gif)


If I implement it in traditional way, who should I notice after I update user's name and save? 
I need to save into database of course, and I need figure out a way to let the icon know my name changes. Maybe write a function called `this.refreshAll()`? oh this is so JQuery!
And I'm sure after deliever the story, we will have a bug report because I forget to update the name in Nav Bar!

Luckily it's 2018 and we have  RxJS, I just need to update the `Observable<User>`, I know every places which need the User should all subscribed to this Observable, and everything should automatically updated. 
Ahh, Inner peace.  


### Async? 
```
Javascript is async but not concurrent
                        - Chris Penner
                          Nova Scotia, 1867
```

Async is good, but dangers sometimes. We already have sons of RxJS code in our projects. so instead of showing what is correct, I prefer to show something includes mistakes, and let's correct them together.

```javascript
this.currentUser$ = this.userService.currentUser;
this.currentUser$.subscribe((currentUser) => {
    this.userProfileForm = new FormGroup({
        firstName: new FormControl(currentUser.firstName),
        lastName: new FormControl(currentUser.lastName),
        email: new FormControl({value: currentUser.email, disabled: true}, Validators.email),
        phone: new FormControl(currentUser.phone)
    });
});
```

Basically what this code doing subscribe to an RxJS Observable this.currentUser$, and use the value to create an FormGroup object and assign it to this.userProfileForm.

It looks maybe Okay, right? 

okay let's see another story. 

> I'm going to make a reservation in a restaurant and have dinner with my friends. 

> I called the restaurand and ask if I can make a reservation tonight.

> The receptionist told me they are not sure, they don't know so they will call me back later.

> I called my friends and tell them our reservation time is `undefined` o'clock

> The restaurand called me back and tell me the are available at 6pm. 

> I had dinner alone, all of my friends can't stop crying and crashing because don't know how to handle `undefined`.

Now you see the problem.  Same in the example code, when we use the `this.userProfileForm`, we can't guarantee the value is already assigned!

A proper way to fix it is set the userProfileForm to Observable, then 

```javascript
this.userProfileForm$ = this.currentUser$.map(currentUser => {
  return new FormGroup({
    firstName: new FormControl(currentUser.firstName),
    lastName: new FormControl(currentUser.lastName),
    email: new FormControl(
      { value: currentUser.email, disabled: true },
      Validators.email
    ),
    phone: new FormControl(currentUser.phone)
  });
});

```

When use the userProfileForm, instead of use the value directly, `Subscribe` it and do the action in side the subscribe. It will waiting until there is a `FormGroup` come in.


## Summary
`undefined`


## Next time we will talk about
- Observer/Observable/Subject
- different operator, when to use them, and how to use them
- Common mistakes
