# RxValidator
Simple, Extensable, Flexable Validation Checker

## Requirements

`RxValidator` is written in Swift 4. Compatible with iOS 8.0+

## Installation

### Cocoapods

RxValidator is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod 'RxValidator'
```

## Usage

You just use like this:
```swift

Validate.to(TargetValue)
    .validate(condition)
    .validate(condition)
    .validate(condition)
        ...
    .validate(condition)
    .asObservable() or .check()
    
```

### String
#### Use RxSwift
```swift {.line-numbers}
	
Validate.to("word is not empty")
    .validate(StringIsShouldNotEmpty())
    .asObservable()
    .subscribe(onNext: { value in
        print(value)
	//print("word is not empty")
    })
    .disposed(by: disposeBag)

Validate.to("word is not empty")
    .validate(StringIsShouldNotEmpty())
    .asObservable()
    .map { $0 + "!!" }
    .bind(to: anotherObservableBinder)
    .disposed(by: disposeBag)
	

//Multiple condition
Validate.to("vbmania@me.com")
    .validate(StringIsShouldNotEmpty())                         //(1)
    .validate(StringIsNotOverflowThen(maxLength: 50))           //(2)
    .validate(StringIsShouldMatch("[a-z]+@[a-z]+\\.[a-z]+"))    //(3)
    .asObservable()
    .subscribe(onNext: { value in
        print(value)
        //print("vbmania@me.com")
    },
    onError: { error in
        let validError = RxValidatorErrorType.determine(error: error)
        // (1) validError -> RxValidatorErrorType.stringIsEmpty
        // (2) validError -> RxValidatorErrorType.stringIsOverflow
        // (3) validError -> RxValidatorErrorType.stringIsNotMatch
    })
    .disposed(by: disposeBag)
		
```

#### Pure Swift
```swift {.line-numbers}
	
Validate.to("word is not empty")
    .validate(StringIsShouldNotEmpty())
    .check()
// result -> RxValidatorErrorType.valid

//multiple condition
Validate.to("vbmania@me.com")
    .validate(StringIsShouldNotEmpty())
    .validate(StringIsNotOverflowThen(maxLength: 50))
    .validate(StringIsShouldMatch("[a-z]+@[a-z]+\\.[a-z]+"))
    .check()
// result -> RxValidatorErrorType.valid

```

### Date
#### Use RxSwift
```swift {.line-numbers}

let targetDate: Date //2018-05-05
let sameTargetDate: Date
let afterTargetDate: Date
let beforeTargetDate: Date

Validate.to(Date())
	.validate(.shouldEqualTo(date: sameTargetDate))             //(1)
	.validate(.shouldAfterOrSameThen(date: sameTargetDate))     //(2)
	.validate(.shouldBeforeOrSameThen(date: sameTargetDate))    //(3)
	.validate(.shouldBeforeOrSameThen(date: afterTargetDate))   //(4)
	.validate(.shouldBeforeThen(date: afterTargetDate))         //(5)
	.validate(.shouldAfterOrSameThen(date: beforeTargetDate))   //(6)
	.validate(.shouldAfterThen(date: beforeTargetDate))         //(7)
	.asObservable()
	.subscribe(onNext: { value in
        print(value) //print("2018-05-05")
	}, onError: { error in
		let validError = RxValidatorErrorType.determine(error: error)
		
        // (1) validError -> RxValidatorErrorType.notEqualDate
        // (2) validError -> RxValidatorErrorType.notAfterDate
        // (3) validError -> RxValidatorErrorType.notBeforeDate
        // (4) validError -> RxValidatorErrorType.notBeforeDate
        // (5) validError -> RxValidatorErrorType.notBeforeDate
        // (6) validError -> RxValidatorErrorType.notAfterDate
        // (7) validError -> RxValidatorErrorType.notAfterDate
	})
	.disposed(by: disposeBag)

```

#### Pure RxSwift
```swift {.line-numbers}

let targetDate: Date //2018-05-05
let sameTargetDate: Date
let afterTargetDate: Date
let beforeTargetDate: Date

Validate.to(Date())
	.validate(.shouldEqualTo(date: sameTargetDate))             //(1)
	.validate(.shouldAfterOrSameThen(date: sameTargetDate))     //(2)
	.validate(.shouldBeforeOrSameThen(date: sameTargetDate))    //(3)
	.validate(.shouldBeforeOrSameThen(date: afterTargetDate))   //(4)
	.validate(.shouldBeforeThen(date: afterTargetDate))         //(5)
	.validate(.shouldAfterOrSameThen(date: beforeTargetDate))   //(6)
	.validate(.shouldAfterThen(date: beforeTargetDate))         //(7)
	.check()
	
	// check() result
	
	// valid result  -> RxValidatorErrorType.valid
	
	// (1) not valid -> RxValidatorErrorType.notEqualDate
	// (2) not valid -> RxValidatorErrorType.notAfterDate
	// (3) not valid -> RxValidatorErrorType.notBeforeDate
	// (4) not valid -> RxValidatorErrorType.notBeforeDate
	// (5) not valid -> RxValidatorErrorType.notBeforeDate
	// (6) not valid -> RxValidatorErrorType.notAfterDate
	// (7) not valid -> RxValidatorErrorType.notAfterDate

```


### Int

not Implementation


### ResultType
```swift {.line-numbers}
enum RxValidatorResult

    case valid
    case notValid(code: Int)
    
    case undefinedError
    
    case stringIsOverflow
    case stringIsEmpty
    case stringIsNotMatch
    
    case invalidateDateTerm
    case notBeforeDate
    case notAfterDate
    case notEqualDate
```

### Use with ReactorKit (http://reactorkit.io)
```swift {.line-numbers}
func mutate(action: Action) -> Observable<Mutation> {
....

case let .changeTitle(title):
  return Validate.to(title)
    .validate(StringIsNotOverflowThen(maxLength: TITLE_MAX_LENGTH))
    .asObservable()
    .flatMap { Observable<Mutation>.just(.updateTitle(title: $0)) }
    .catchError({ (error) -> Observable<Mutation> in
        let validError = ValidationTargetErrorType.determine(error: error)
        return Observable<Mutation>.just(.setTitleValidateError(validError, title))
    })

....

```


## If you want make custom ValidationRule write code like below:
```swift
//String Type

class MyCustomStringValidationRule: StringValidatorType {
    func validate(_ value: String) throws {
        if {notValidCondition} {
            throw RxValidatorResult.notValidate(code: 999) //'code' must be defined your self.  
        }
    }
}


//Int Type

class MyCustomIntValidationRule: IntValidatorType {
    func validate(_ value: Int) throws {
        if {notValidCondition} {
            throw RxValidatorResult.notValidate(code: 999) //'code' must be defined your self.  
        }
    }
}


```



## What's next...

* Int Valdation Rules
* validate begin from Observable

