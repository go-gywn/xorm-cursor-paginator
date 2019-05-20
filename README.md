xorm-cursor-paginator
=====================

A paginator doing cursor-based pagination based on [XORM](http://xorm.io), which Forked from gorm-cursor-paginator(https://github.com/pilagod/gorm-cursor-paginator) to use with xorm.

Installation
------------

```sh
go get -u github.com/go-gywn/xorm-cursor-paginator
```

Usage by Example
----------------

Assume there is an optional query struct for paging:

```go
type PagingQuery struct {
    AfterCursor     *string
    BeforeCursor    *string
    Limit           *int
    Order           *string
}
```

and a XORM model:

```go
type Model struct {
    ID          int64     `xorm:"id int(11) pk not null autoincr"`
    Name        string    `xorm:"name varchar(30) not null"`
    CreatedAt   time.Time `xorm:"datetime not null created"`
}
```

You can simply build up a cursor paginator from the PagingQuery like:

```go
import (
    paginator "github.com/pilagod/gorm-cursor-paginator"
)

func InitModelPaginatorFrom(q PagingQuery) paginator.Paginator {
    p := paginator.New()

    p.SetKeys("CreatedAt", "ID") // [defualt: "ID"] (order of keys matters)

    if q.AfterCursor != nil {
        p.SetAfterCursor(*q.AfterCursor) // [default: ""]
    }

    if q.BeforeCursor != nil {
        p.SetBeforeCursor(*q.BeforeCursor) // [default: ""]
    }

    if q.Limit != nil {
        p.SetLimit(*q.Limit) // [default: 10]
    }

    if q.Order != nil {
        if *q.Order == "ascending" {
            p.SetOrder(paginator.ASC) // [default: paginator.DESC]
        }
    }
    return p
}
```

Then you can start to do pagination easily with XORM:

```go
func Find(xorm *xorm.Engine, q PagingQuery) ([]Model, paginator.Cursors, error) {
    var models []Model

    session := xorm.NewSession()
    session = session.Where("name = ?", "chan")
    session = session.Where(/* ... more other filters ... */)

    // init paginator for Model
    p := InitModelPaginatorFrom(q)

    // use XORM-like syntax to do pagination
	p := paginator.New()
	if error := p.Paginate(session, &models); error != nil {
		panic(error)
	}
    
    // get cursors for next iteration
    cursors := p.GetNextCursors()

    return models, cursors, nil
}
```

After pagination, you can call `GetNextCursors()`, which returns a `Cursors` struct, to get cursors for next iteration:

```go
type Cursors struct {
    AfterCursor     string
    BeforeCursor    string
}
```

That's all ! Enjoy your paging in the XORM world :tada:

License
-------
© Dong-chan Sung (go-gywn), 2019-NOW

Released under the [MIT License](https://github.com/go-gywn/xorm-cursor-paginator/blob/master/LICENSE)
