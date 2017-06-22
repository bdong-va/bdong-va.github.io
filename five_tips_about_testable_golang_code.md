# How to write UNtestable code in golang
According to my experience in movies, if you want to be a super bad guy, you will need to work harder than the good guys. Think about it! You need to hire evil scientists, build a huge death bomb to threaten the world, a good plan and a lot of money. But be a good guy you just basically need some brave hearts, you do not even have to know about the structure of bomb because when choosing red/blue wire to cut, good guys never mess up.

Yeah, that's not fair.

Good news (?) everyone, in software developer side, mess up code is far easier than build it in a good way. If you keep following points in mind, make golang code untestable would be so easy.

Before reading this, I assume you already read [Aaron's Blog: Testify - A Toolkit for Golang Unit Tests](https://vendasta-blog.appspot.com/blog/BL-F5QKDT6H/).

## 1. Always expected golang mock as powerful as Python mock

This is the easiest part. If you start with something with Python, all you need is ~~love~~ do nothing extra, just write code with all Python style, you can make it untestable.

First, let take a look at this chart:

|        | python mock | golang testify mock |
|--------|-------------|-------------|
|mock a class|  Yes|         Yes    |
|mock a method in a mock class| Yes |Yes|
|assert mock been called with given parameter | Yes | Yes|
|return any result in mock method | Yes |Yes|
|mock an attribute in a mock class | Yes | No |
|mock a function | Yes | No|
| patch | Yes | No |

![that is how good Python is.](https://imgs.xkcd.com/comics/python.png)

Mock(testify) in golang is good. Just not quite as *magical* as mocking in Python. When I first learn how to use it, the thought occurred to me most frequently was "What, I need to do this manually?" and "If I need to do this manually, what are YOU doing, mock?"

~~" I am Groot." Mock replied.~~

Mock in python can automatically do two things, one is **replace current real code to mock code**, another is **verify calls happen as expected**.

Mock in go testify can do the second job(verify calls) but not first one(replace real code to mock). That part has to be done manually. And manually replacement have more limitations than magic replacement.

We can talk about the limitations in next sections, for this section, just simply forget golang mock is not as powerful as python's mock, is a good start to write untestable code in go.

## 2. Interface who? we just pass struct itself.
TL;DR, `struct` cannot be mocked when passing as a parameter.

Let's start with a golang example:
```go
func addAggregation(search *elastic.SearchService) *elastic.SearchService {
    search.Aggregation(/* put some variable here*/)
    search.Aggregation(/*put some other variable here*/)

    return search
}
```

I removed some information to make it more clear. This function takes an `elastic.SearchService`, do something to it and return `elastic.SearchService`.

Test a function like that in Python is easy. Define `mock.Aggregation` somewhere, pass this mock in, assert it be called with given parameter, then assert mock been returned. done.

But we cannot do the same thing in golang.

Remember, golang is a strong type language. If a function takes `elastic.SearchService` as the parameter, you cannot pass anything else. The only possible way is to create a real `elastic.SearchService` object, and do some tweaking in `Aggregation` method. It is complex, and when you start seeking out the solution for "how to create a real elastic.SearchService object", it is easy to get lost in chasing infinite rabbits in infinite holes. You will forget what you want at first. ~~Just like a bad marriage.~~

Good guys have a solution for this. Instead of a struct, passing an interface can solve it.
Still same function:
```go
func addAggregation(search *SearchServiceInterface) *SearchServiceInterface {
    search.Aggregation(/* put some variable here*/)
    search.Aggregation(/*put some other variable here*/)

    return search
}

type SearchServiceInterface interface {
    Aggregation(/* exactly same variable here*/)
}
```

It looks almost the same, the only difference is instead of `elastic.SearchService`, we passed in an `SearchServiceInterface`. For this example, the compiler would not complain about the object passed in if it has `Aggregation(/* correct type variable here*/)` method. that means we can pass a mock like this into it for testing:

```go
type SearchServiceMock struct {
    mock.Mock
}
(ssm *SearchServiceMock) Aggregation(/* exactly same variable here*/) {
    args := ssm.Called(/* exactly same variable here*/)
    return args.Get(0).(*SearchServiceMock)
}
```
Then we can have all the benefits of strong type language and still keep it flexible enough for testing.

Of course, as a bad guy who wants to make golang code untestable, you will just pass a struct into the function, nailed it.

## 3. write function instead of method
TL, DR; Function is not mockable.

Let's start with example:

```go
func (s *scoreRepository) Create(ctx context.Context, accountGroupID string) (int64, error) {
    listingPointScore, err := calculateListingScore(ctx, accountGroupID)

    /*
        some logic we want to test
    */

    return int64(listingPointScore), nil
}
```

If same logic happens in Python, testing will be easy. `@patch` the `calculateListingScore` function and let it return `any_number_we_need, nil`, then verify the result.

But in the cruel world of golang, everything is harder: we do not have `@patch` like we do in Python, and we cannot mocking a function at all. If you call a function in somewhere, you cannot mock it at all. If you want to test a function/method includes an external function call, you have to call that function for real, and That is not good in the world of testing.

So we have a solution for this: put the function into a struct can be mocked by the interface.

```go
func (s *scoreRepository) Create(ctx context.Context, calculator calculatorInterface, accountGroupID string) (int64, error) {
    listingPointScore, err := calculator.calculateListingScore(ctx, accountGroupID)

    /*
        some logic we want to test
    */

    return int64(listingPointScore), nil
}
```

Now when we test it, we can pass a `calculatorMock` which implement with the `calculatorInterface`. Then we can do assertion or make the mock method return whatever value we want.

Or you can choose another way to do it, make the `calculator` an attribute of `scoreRepository`:

```go
type scoreRepository struct {
    /*something impartant
    but not related with
    our currect topic*/
    calculator    *ListingScoreCalculator
}

func (s *scoreRepository) Create(ctx context.Context, accountGroupID string) (int64, error) {
    listingPointScore, listingComponents, citationComponents, err := s.calculator.calculateListingScore(ctx, accountGroupID)


    /*
        some logic we want to test
    */

    return int64(listingPointScore), nil
}
```

The benefit of this way is you can keep the `Create` method signature clean. sometime grpc will require you keep the same signature, or if you just do not want to pass a calculator into a method.

Of course, as a bad guy who wants to mess up golang code to make it untestable, you can just use function everywhere, and enjoy the hopeless face of next developer who will write test in your code.

## 4. TDD? never heard of it.
For Python, almost everything is testable, mock is too magic, we can always have good test coverage for all code style. In this case, I will say TDD is still good to have but not logically required for good code. You can always find a way to test your code after everything works.

But for Golang, TDD have huge benefits on writing testable code. Due to the limitation of strong type code style, it is so easy to write totally worked but almost not testable code. If we do not tighten the fine string of testable in our brain, the refactoring of code to make everything tested will be painful. Not only for the developer who write it, also the one who maintain it in future. ~~and it could be you!~~

If TDD is too hard for some code, just keep the testable idea in mind would still help a lot. After we write a method name and start to pass things in, there should be a tiny checklist in the corner of your brain somewhere, like:
- Is it testable to pass this thing in?
- Is there any function call I cannot mock here?
- How can we reuse the mock which already existed?


## After all...

Fine, let's be honest. I am a bad liar and this article is actually about how to make golang code **testable**.
Let's review the points in **right way**:
1. Remember golang mock is weaker than python mock.
2. Pass interface instead of a struct itself in to make method/function easier to test.
3. Methods are mockable, functions are not.
4. TDD is good. Always keep alert about the code you are writing, make sure they are testable.

