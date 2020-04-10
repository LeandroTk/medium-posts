# Fun with Dates

As an ex-Rubyist, I always liked to work with Ruby dates (mental note: not the timezone part). I liked the human way on how Ruby and Rails provide methods to handle the Date object.

In Ruby, we can get the current date by doing:

```ruby
require 'date'

Date.today # #<Date: 2020-04-05 ((2458945j,0s,0n),+0s,2299161j)>
```

This is pretty cool! I can send a simple message to the Date object "hey, provide me the `today date`" by calling the `today` method.

Or simply get the `year`, `month`, `day`.

```ruby
date = Date.today
date.year # 2020
date.month # 4
date.day # 5
```

Using Rails, it is also possible to call the `yesterday` method.

```ruby
Date.yesterday
```

Rails also provides other interesting APIs: `beginning_of_month`, `minutes.ago`, `days.ago`.

So after a long time with Ruby and Rails, I started using JavaScript more and more. But the JavaScript Date object was really strange for me. I wanted to use all the Ruby/Rails date APIs, but in JavaScript and Typescript.

I didn't want to monkey patch or build new methods in the JavaScript Date object. I could just provide some simple functions and handle the date internally.

## Dating dates

First things first: I wanted to better understand the Date object. How do we create it?

```typescript
new Date();
```

By simply instantiate the Date object. We get the representation of `now` (the current date).

The other APIs I need to try was: `getDate`, `getMonth`, and `getFullYear`. These are all methods to handle the date.

```typescript
const day: number = now.getDate(); // 5
const month: number = now.getMonth(); // 3
const year: number = now.getFullYear(); // 2020
```

We could experiment a whole bunch of other methods here, but I think we are good to move to the next part.

## Fun with dates

In this part, we will build functions! I wanted to try creating this API:

- day
- month
- year
- today
- yesterday
- beginningOfDay
- beginningOfMonth
- beginningOfYear
- get(1).dayAgo
- get(2).daysAgo
- get(1).monthAgo
- get(2).monthsAgo
- get(1).yearAgo
- get(2).yearsAgo

## day, month, and year

In this case, we provide a date and it will return the day of this date we provided.

```typescript
const day = (date: Date): number => date.getDate();
const month = (date: Date): number => date.getMonth();
const year = (date: Date): number => date.getFullYear();
```

And we can use it like:

```typescript
const now = new Date();

day(now); // 5
month(now); // 3
year(now); // 2020
```

## today and yesterday

With `today` function, we could just return the `new Date()` and we are good. But this returns the representation of `now` with "time" included.

```typescript
new Date(); // 2020-04-05T18:58:45
```

But it would be great to return the beginning of the day. We could simply pass the day, month, and year to the `Date` and it will generates this for us.

```typescript
const today = (): Date => {
  const now: Date = new Date();
  const day: number = now.getDate();
  const month: number = now.getMonth();
  const year: number = now.getFullYear();

  return new Date(year, month, day);
};
```

Great. The `yesterday` function would work very similiar. Just subtract the day and we are good to go.

```typescript
const yesterday = (): Date => {
  const now: Date = new Date();
  const day: number = now.getDate();
  const month: number = now.getMonth();
  const year: number = now.getFullYear();

  return new Date(year, month, day - 1);
};
```

But what happen when we subtract the day if the day is the first day of the month?

```typescript
// date to handle
new Date(2020, 3, 1); // 2020-04-01

// when subtracting the day: from 1 to 0
new Date(2020, 3, 0); // 2020-03-31
```

And what happen if it is the first day of the year?

```typescript
// date to handle
new Date(2020, 0, 1); // 2020-01-01

// when subtracting the day: from 1 to 0
new Date(2020, 0, 0); // 2019-12-31
```

Yes, JavaScript can be pretty smart too!

With these two new functions, we can also refactor the logic to get the separated date into a separated function.

```typescript
const getSeparatedDate = (): { day: number, month: number, year: number } => {
  const now: Date = new Date();
  const day: number = now.getDate();
  const month: number = now.getMonth();
  const year: number = now.getFullYear();

  return { day, month, year };
};
```

Let's improve this! This returned type could be a Typescript `type`.

```typescript
type SeparatedDate = {
  day: number
  month: number
  year: number
};
```

Less verbose now:

```typescript
const getSeparatedDate = (): SeparatedDate => {
  const now: Date = new Date();
  const day: number = now.getDate();
  const month: number = now.getMonth();
  const year: number = now.getFullYear();

  return { day, month, year };
};
```

I this case, we are always returning the `day`, `month`, and `year` attributes of the current date. But what if we want to pass a different date? A new argument to the rescue:

```typescript
const getSeparatedDate = (now: Date = new Date()): SeparatedDate => {
  const day: number = now.getDate();
  const month: number = now.getMonth();
  const year: number = now.getFullYear();

  return { day, month, year };
};
```

Now we have a function that can receive a new date, but if it doesn't, it just use the default value: the representation of `now`.

How does our functions `today` and `yesterday` look like now?

```typescript
const today = (): Date => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  return new Date(year, month, day);
};

const yesterday = (): Date => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  return new Date(year, month, day - 1);
};
```

Both functions use the `getSeparatedDate` function to get the date attributes and return the appropriate date.

## Beginning of everything

To build the `beginningOfDay`, it would look exatcly of the `today` function, as we want to the current date but at the beginning of the day.

```typescript
const beginningOfDay = (): Date => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  return new Date(year, month, day);
};
```

Nothing special here.

For the `beginningOfMonth`, it will look like pretty much the same, but instead of using the `day`, we just set it to `1`.

```typescript
const beginningOfMonth = (): Date => {
  const { month, year }: SeparatedDate = getSeparatedDate();

  return new Date(year, month, 1);
};
```

You got it, the `beginningOfYear` is similar. But also changing the `month` attribute.

```typescript
const beginningOfYear = (): Date => {
  const { year }: SeparatedDate = getSeparatedDate();

  return new Date(year, 0, 1);
};
```

## Traveling back in time

Now the `get(1).dayAgo` API. We could build a `get` function that receives a `number` and return an object like:

```typescript
{
  dayAgo,
  monthAgo,
  yearAgo
}
```

For each attribute of this object, it would be the returned value we expect.

```typescript
const get = (n: number): { dayAgo: Date, monthAgo: Date, yearAgo: Date } => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  const dayAgo: Date = new Date(year, month, day - n);
  const monthAgo: Date = new Date(year, month - n, day);
  const yearAgo: Date = new Date(year - n, month, day);

  return { dayAgo, monthAgo, yearAgo };
};
```

What about a `DateAgo` type?

```typescript
type DateAgo = {
  dayAgo: Date
  monthAgo: Date
  yearAgo: Date
};
```

And now using the new type:

```typescript
const get = (n: number): DateAgo => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  const dayAgo: Date = new Date(year, month, day - n);
  const monthAgo: Date = new Date(year, month - n, day);
  const yearAgo: Date = new Date(year - n, month, day);

  return { dayAgo, monthAgo, yearAgo };
};
```

We build each attribute: `dayAgo`, `monthAgo`, and `yearAgo` by basically handling the Date object as we know.

But now we also need to implement the object in the plural: `daysAgo`, `monthsAgo`, and `yearsAgo`. But only for number greater than 1.

For these new attributes, we don't need to create a whole new date again. We can use the same value from the singular attributes.

We also need to handle the `number` received.

- if it is greater than 1: return the object with plural attributes
- otherwise: return the object with singular attributes

```typescript
const get = (n: number): DateAgo | DatesAgo => {
  const { day, month, year }: SeparatedDate = getSeparatedDate();

  const dayAgo: Date = new Date(year, month, day - n);
  const monthAgo: Date = new Date(year, month - n, day);
  const yearAgo: Date = new Date(year - n, month, day);

  const daysAgo: Date = dayAgo;
  const monthsAgo: Date = monthAgo;
  const yearsAgo: Date = yearAgo;

  return n > 1
    ? { daysAgo, monthsAgo, yearsAgo }
    : { dayAgo, monthAgo, yearAgo };
};
```

- In this case, I also created the `DatesAgo` type and used the Typescript `Union Type` feature.
- We reuse the singular values.
- And do a simple ternary to handle the number received.

But what if we pass a `0` or negative value? We can throw an error:

```typescript
const get = (n: number): DateAgo | DatesAgo => {
  if (n < 1) {
    throw new Error('Number should be greater or equal than 1');
  }

  const { day, month, year }: SeparatedDate = getSeparatedDate();

  const dayAgo: Date = new Date(year, month, day - n);
  const monthAgo: Date = new Date(year, month - n, day);
  const yearAgo: Date = new Date(year - n, month, day);

  const daysAgo: Date = dayAgo;
  const monthsAgo: Date = monthAgo;
  const yearsAgo: Date = yearAgo;

  return n > 1
    ? { daysAgo, monthsAgo, yearsAgo }
    : { dayAgo, monthAgo, yearAgo };
};
```

The Date can be fun too. Learn the basic concepts and just play around with it, you'll like! I hope this post was valuable to you!

## Resources

- [Date - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [Ruby on Rails Date API](https://api.rubyonrails.org/classes/Date.html)
- [Ruby Date API](https://ruby-doc.org/stdlib-2.7.1/libdoc/date/rdoc/Date.html)
- [Dating library](https://github.com/leandrotk/dating)
- [Typescript Learnings 001: Object Destructuring](https://leandrotk.github.io/tk/2020/04/typescript-learnings/001-object-destructuring.html)
- [Understanding Date and Time in JavaScript](https://www.digitalocean.com/community/tutorials/understanding-date-and-time-in-javascript)
