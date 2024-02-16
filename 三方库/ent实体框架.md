

ent是一款便于操作的orm框架

```shell
go get entgo.io/ent/cmd/ent
```

创建schema

```
ent new --target example/ent/internal/data/schema Car Group
```

生成代码

生成注释：

```go
//go:generate go run entgo.io/ent/cmd/ent generate ./schema --feature sql/lock --template ../../../../schema/templates
```

生成命令

```
go generate ./...
```

## ent使用

Ent  接口定义

```go
	// The Interface type describes the requirements for an exported type defined in the schema package.
	// It functions as the interface between the user's schema types and codegen loader.
	// Users should use the Schema type for embedding as follows:
	//
	//	type T struct {
	//		ent.Schema
	//	}
	//
	Interface interface {
		// Type is a dummy method, that is used in edge declaration.
		//
		// The Type method should be used as follows:
		//
		//	type S struct { ent.Schema }
		//
		//	type T struct { ent.Schema }
		//
		//	func (T) Edges() []ent.Edge {
		//		return []ent.Edge{
		//			edge.To("S", S.Type),
		//		}
		//	}
		//
		Type()
		// Fields returns the fields of the schema.
		Fields() []Field
		// Edges returns the edges of the schema.
		Edges() []Edge
		// Indexes returns the indexes of the schema.
		Indexes() []Index
		// Config returns an optional config for the schema.
		//
		// Deprecated: the Config method predates the Annotations method, and it
		// is planned be removed in v0.5.0. New code should use Annotations instead.
		//
		//	func (T) Annotations() []schema.Annotation {
		//		return []schema.Annotation{
		//			entsql.Annotation{Table: "Name"},
		//		}
		//	}
		//
		Config() Config
		// Mixin returns an optional list of Mixin to extends
		// the schema.
		Mixin() []Mixin
		// Hooks returns an optional list of Hook to apply on
		// the executed mutations.
		Hooks() []Hook
		// Interceptors returns an optional list of Interceptor
		// to apply on the executed queries.
		Interceptors() []Interceptor
		// Policy returns the privacy policy of the schema.
		Policy() Policy
		// Annotations returns a list of schema annotations to be used by
		// codegen extensions.
		Annotations() []schema.Annotation
	}

```



### Mixin() []Mixin

`Mixin` 是一种用于将通用功能和字段添加到模型中的机制。Mixin 是一种抽象，允许你将一组字段或方法组合到一个模型中，以便在多个模型之间共享或复用这些功能。

例如，在User模型中添加通用的`mixin.Time{}` 时间类型，默认会添加`create_time`和`update_time`

```go
func (u User) Mixin() []ent.Mixin {
	return []ent.Mixin{
		mixin.Time{},
	}
}
```

mixing.Time定义：

```go
// Time composes create/update time mixin.
type Time struct{ Schema }

// Fields of the time mixin.
func (Time) Fields() []ent.Field {
	return append(
		CreateTime{}.Fields(),
		UpdateTime{}.Fields()...,
	)
}
```

在实际项目中应该还有软删除功能，就是只记录删除时间，而不真正删除，这里自定义`Time`

```go

type DeleteTime struct {
	ent.Schema
}

var _ ent.Mixin = (*DeleteTime)(nil)

func (DeleteTime) Fields() []ent.Field {
	return []ent.Field{
		field.Time("delete_time").Optional().Nillable().UpdateDefault(time.Now()),
	}
}

// 自定义Time包含了创建、更新、删除时间
type Time struct{ ent.Schema }

func (Time) Fields() []ent.Field {
	return append(DeleteTime{}.Fields(), mixin.Time{}.Fields()...)
}

```



### Annotations() []schema.Annotation