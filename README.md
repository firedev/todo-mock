# Todo Mockup

```js
task = {
  id: uuid.required,
  title: string.required,
  completed: boolean.required,
  priority: oneOf([1, 2, 3]).required,
  description: string, // markdown, multi-line
  dueDate: string, // date-like string
  sheduleDate: string, // same
  // optional
  position: number, // if time permits so one can rearrange cards
}

task.defaultProps = {
  completed: false,
  priority: 1,
}
```

```css
/** colors **/
--red: #e80d00;
--orange: #ff851b;
--blue: #7fdbff;
```

```js
PRIORITIES = {
	1: { title: 'P1', color: '--red' }
	2: { title: 'P2', color: '--orange' }
	3: { title: 'P3', color: '--blue' }
}
```

# Stack

- https://github.com/facebook/react - for components
- https://github.com/apollographql/react-apollo - for data loading
- https://github.com/atlassian/react-beautiful-dnd for drag and drop
- https://github.com/formium/formik - forms
- https://github.com/jquense/yup - validation
- https://github.com/datejs/Datejs - human date parser
- https://github.com/wojtekmaj/react-calendar - dropdown calendar

# Interface

- The app loads all tasks at once and sorts them once they are loaded.
- This way we don't need to worry about having additional models on the backend.
- If we hide the `Done` tasks- we can load only incomplete tasks.
- This is debateable but shouln't be a problem unless someone creates billion cards, and we can test things sooner.

```gql
{ tasks { id title ... }}
```

- When creating/editing a task we display all fields, but pre-fill `scheduledDate` depending on a column.
- Users can change data and the card could end up in a different column after save.
- New card goes to the top of the list so it's not confusing
- On the mobile phone we see only one column, so the app shows the resulting column the card is added to.
- I propose `apollo-graphql` with optimistic updates turned on, so after validation in the frontend we can be almost sure the actual data will be correct and update interface right away.
- Generate `uuid` in the frontend as well
- When one starts dragging a card on the mobile phone, replace the column view with slots to drop the card into:

```
+----------+
| Backlog  |
+----------+
| Today    |
+----------+
| Tomorrow |
+----------+
| Schedule |
+----------+
| Done     |
+----------+
```

- When card needs to be scheduled use an additional route to display the calendar `/tasks/id/schedule` this way the state is easy to manage and until the user enters the date the card stays where it was
- When creating a card in scheduled column without the date we should drop it to the `Backlog` and let user decide, if time allows - add an `Are you sure?` dialog
- Cards are split per column by their `id` and if time permits – `position`:

```js
state: {
  cards: {
    uuid10: { id: 'uuid10', column: 'backlog', title: 'Hello' },
    uuid11: { .... },
    ...
  },
  columns: {
    backlog: [ uuid10, uuid11, ... ],
    today: [ uuid20, uuid21, ... ],
    tomorrow: [ uuid30, uuid31, ... ],
    scheduled: [ uuid40, uuid41, ... ],
    done: [ uuid50, uuid51, ... ],
  }
},
```

# `TaskCard`

Component that displays `task`

![task.png](task.png)

- `title` is always displayed in full
- `description` is cut off so it's not too long, rendered using markdown
- `priority` is color coded
- `completed` fades out priority 50% and displays the `completed` icon in the corner
- `sheduleDate` and `dueDate` are displayed without the year if the year is current
- Human-readable time to/from each of the dates
- If both `sheduleDate` and `dueDate` are present display human-readable estimates amount of time
- When there is not enough width date lines go one under another, similar to [task-create-edit.png](task-create-edit.png)
- Fluid width, no maximum width, defined by the column container
- Draggable

# `CardColumn`

Component that displays a stack of `TaskCard` + `AddTask`, see [column.png](column.png) and [board.png](board.png)

![](column.png)

- On mobile we display only one column and column title becomes a dropdown menu to switch between columns
- Clicking `AddTask` opens `TaskForm` component, same when you click the card
- When dragging the card on the desktop one sees the marker where the card was dragged from and where it lands
- EXTRA: In the first version cards are auto-sorted, later we could add `position`. Then cards will be ordered by `position` which is scoped to a column. New card will have `position: 0`, so it is always at the top.

# `TaskBoard`

One can use board as simple scheduler, cards with no near dates, today, tomorrow, and due later.

- Cards are auto-sorted based on `scheduledDate` and `completed` using the following rules, they are applied top to bottom: - no `scheduledDate` - `Backlog` - `scheduledDate` is `today` - `Today` - `scheduledDate` is `tomorrow` - `Tomorrow` - `scheduledDate` exists and not `[today, tomorrow]` - `Scheduled` - `completed`, with any dates – `Done`
- Dragging card from one column to another changes it's `scheduledDate`, and sets/clears `completed` if applicable: - `Backlog` - clear `scheduledDate`, `completed: false` - `Today` - set `scheduledDate` to `today`, `completed: false` - `Tomorrow` - set `scheduledDate` to `tomorrow`, `completed: false` - `Scheduled` - set `completed: false`, display the date popup if `scheduledDate` is `[ null, today, tomorrow ]`
- Due date can only be changed manually

# `TaskForm`

This is what the user sees when they click the card or `Add task`

![](task-create-edit.png)

- `Add task` prefills `scheduledDate` depending on a column:
  - `Backlog` - nothing is prefilled
  - `Today` - `scheduledDate` is set to today
  - `Tomorrow` - `scheduledDate` is set to tomorrow
  - `Scheduled` - `scheduledDate` is not set but when you try to save you are shown a warning and you can change the date or the card lands in `Backlog`. If you enter today's or tomorrow's date, the card lands in the respective column.
- Card is highlighted after creation so it can be easily seen.
- On the mobile the view switches to the column the card is created in
  - The simplest way to do this is via `CSS`, so the last column has `.active` class and the other ones are hidden when the viewport is too small.
- They can enter dates manually or via dropdown once they focus in a field
- Manual entry is very fast on the desktop as you can just type `13` or `monday` and it will be recognized.
- By default priority is `P3` (low)
- Card can be completed right away then it will be added on the top of the `Done` column

# Concerns

- `dueDate` does not affect anything, we should at least highlight overdue cards
- `startDate` is a better way for `scheduledDate` should be named
- Original version did not have `Done` column but I see no reason keeping the `Done` items in sight. We could even hide the `Done` column and treat is as an `Archive` if we need to restore something
- Task splitting could be done on the backend as well

# Estimates

`3` is a standard task, roughly half a day's work.

## Day 1 - Task Card

- TaskCard component - 5

  - Responsive styles
  - Subcomponents for priority, dates

- Data loading and splitting - 3

## Day 2 - Task Column

- TaskColumn component - 5

  - Display Cards nicely
  - Add title
  - Add dropdown for on mobile

- Add task component - 3

## Day 3 - Task Board

- Board layout - 2

  - Display multiple columns

- Add drag-n-drop - 3

  - Add placeholders

- Card form - 5

  - Create/edit card
  - Card created with no date
  - Card created for today
  - Card created for tomorrow

## Day 4 - Card Create

- Scheduled card creation - 5

  - Scheduler popup
  - Calendar

- Make sure unscheduled Tasks display popup when dropped in the schedules column - 3

## Day 5 - Done Column

- Done column - 3

  - Convert cards to Done when dropped

- EXTRA: Positioning - 5

# Figures

![](task.png)
![](task-grid.png)
![](task-create-edit.png)
![](column.png)
![](board.png)
