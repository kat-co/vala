vala
====

A simple golang library to make argument validation palatable.

Instead of this:

```go
func BoringValidation(a, b, c, d, e, f, g MyType) {
  if (a == nil)
    panic("a is nil")
  if (b == nil)
    panic("a is nil")
  if (c == nil)
    panic("a is nil")
  if (d == nil)
    panic("a is nil")
  if (e == nil)
    panic("a is nil")
  if (f == nil)
    panic("a is nil")
  if (g == nil)
    panic("a is nil")
}
```

Do this:

```go
func ClearValidation(a, b, c, d, e, f, g MyType) {
  BeginValidation().Validate(
    IsNotNil(a, "a"),
    IsNotNil(b, "b"),
    IsNotNil(c, "c"),
    IsNotNil(d, "d"),
    IsNotNil(e, "e"),
    IsNotNil(f, "f"),
    IsNotNil(g, "g"),
  ).CheckAndPanic() // All values will get checked before an error is thrown!
}
```

Instead of this:

```go
func BoringValidation(a, b, c, d, e, f, g MyType) error {
  if (a == nil)
    return fmt.Errorf("a is nil")
  if (b == nil)
    return fmt.Errorf("a is nil")
  if (c == nil)
    return fmt.Errorf("a is nil")
  if (d == nil)
    return fmt.Errorf("a is nil")
  if (e == nil)
    return fmt.Errorf("a is nil")
  if (f == nil)
    return fmt.Errorf("a is nil")
  if (g == nil)
    return fmt.Errorf("a is nil")
}
```

Do this:

```go
func ClearValidation(a, b, c, d, e, f, g MyType) error {
  defer func() { recover() }
  BeginValidation().Validate(
    IsNotNil(a, "a"),
    IsNotNil(b, "b"),
    IsNotNil(c, "c"),
    IsNotNil(d, "d"),
    IsNotNil(e, "e"),
    IsNotNil(f, "f"),
    IsNotNil(g, "g"),
  ).CheckSetErrorAndPanic(&error) // Return error will get set, and the function will return.

  // ...

  VeryExpensiveFunction(c, d)
}
```

Tier your validation:

```go
func ClearValidation(a, b, c MyType) error {
  defer func() { recover() }
  error = BeginValidation().Validate(
    IsNotNil(a, "a"),
    IsNotNil(b, "b"),
    IsNotNil(c, "c"),
  ).CheckAndPanic().Validate( // Panic will occur here if a, b, or c are nil.
    HasLen(a.Items, 50, "a.Items"),
    GreaterThan(b.UserCount, 0, "b.UserCount"),
    Equals(c.Name, "Vala", "c.name"),
    Not(Equals(c.FriendlyName, "Foo", "c.FriendlyName")),
  ).Check()

  // ...

  VeryExpensiveFunction(c, d)
}
```

Extend with your own validators for readability:

```go
func ReportFitsRepository(report *Report, repository *Repository) Checker {
	return func() (bool, error) {
		if !repository.Type == report.Type {
			return false, fmt.Errof("A %s report does not belong in a %s repository.", report.Type, repository.Type)
		}

		return true, nil
	}
}

func AuthorCanUpload(authorName string, repository *Repository) Checker {
	return func() (bool, error) {
		if !repository.AuthorCanUpload(authorName) {
			return false, fmt.Errof("%s does not have access to this repository.", authorName)
		}

		return true, nil
	}
}

func AuthorIsCollaborator(authorName string, report *Report) Checker {
	return func() (bool, error) {

		for _, collaboratorName := range report.Collaborators() {
			if collaboratorName == authorName {
				isValid = true
				break
			}
		}

		if !isValid {
			return false, fmt.Errorf("The given author was not one of the collaborators for this report.")
		}

		return true, nil
	}
}

func HandleReport(authorName string, report *Report, repository *Repository) {

	BeginValidation().
		Validate(AuthorIsCollaborator(authorName, report)).
		Validate(AuthorCanUpload(authorName, repository)).
		Validate(ReportFitsRepository(report, repository)).
		CheckAndPanic()
}
```
