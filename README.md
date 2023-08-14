# Develop in Swift Data Collections

## Unit 1 Tables and Persistence

- In this unit, you'll learn three important sets of techniques in app development, which—taken together—will enable you to build more much complex apps.
  - First, you'll learn how to use tables to display lists of information, and to use the master-detail design pattern to allow users to interact with the information.
  - Second, you'll learn how to organize the files, structures, and classes in your apps, making it easier for you (or other developers) to work with them in the future.
  - Third, you'll learn about an approach for saving data to the device, so that any information the user enters or changes in your app will still be there the next time they open the app.
  - By the end of this unit, you'll be comfortable building many useful apps that display all kinds of information and that allow users to enter, edit, and save in-app information.

### Lesson 1.1 Protocols

- A protocol is a set of rules or procedures that define how things are done. Computers communicate with each other using protocols like HTTP (Hyper Text Transfer Protocol) and TCP/IP (Transmission Control Protocol/Internet Protocol).
  - HTTP is a standard that defines how website data is communicated between two computers.
  - TCP/IP is a communications standard that defines how computers find and send data to each other.
- In programming, a protocol defines the properties or methods that an object must have to complete a task. For example, the `Equatable` protocol says that a type must define a `==` method to check whether two instances are equal to each other.
- In this lesson, you'll learn what protocols are, when to use them, and how to write your own. You'll also learn about delegation, a pattern for enabling objects to communicate with each other.
- A protocol defines a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality. The protocol can then be adopted by another type, which provides the actual implementation for those requirements. Any type that satisfies the requirements of a protocol conforms to that protocol.
- The Swift standard library defines many protocols that you'll use when building apps. In the following sections, you'll look at four of them:
  - `CustomStringConvertible`, which allows you to control how your custom objects are printed to the console;
  - `Equatable`, which allows you to define how instances of the same type are equal to each other;
  - `Comparable`, which allows you to define how instances of the same type are sorted; and
  - `Codable`, which allows you to encode your type's properties as key/value pairs that can then be saved between app launches.
- When you adopt a protocol in Swift, you're promising to implement all the methods required by that protocol. The compiler will check that everything's in order and won't build your program if anything is missing.
- Why might you use protocols when building an app?
  - To share attributes and functionality across different types
  - To make custom types work well with system or debugging functionality
  - To enforce similarity and provide type functionality
  - To define events but delegate implementation to an instance of another type

#### Printing Information with `CustomStringConvertible`

- You can begin to understand the role of a protocol by considering how objects are printed to the console using the `print()` function. As you've learned, running the print() function on a variable will write the textual representation of the object to the console. You can print String values, Int values, and Bool values with predictable results.
- But what if you define a Shoe class?

  - ```swift
      class Shoe {
        let color: String
        let size: Int
        let hasLaces: Bool
       
        init(color: String, size: Int, hasLaces: Bool) {
          self.color = color
          self.size = size
          self.hasLaces = hasLaces
        }
      }
       
      let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
      let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
      print(myShoe)
      print(yourShoe)
      Console Output:
      Shoe
      Shoe
    ```

- (Your print statement is likely prefixed by something like \_lldb_expr_1, which you can ignore.)
- As you can see, this print statement isn't at all helpful. Swift defines a protocol called CustomStringConvertible to address this kind of situation.
- CustomStringConvertible has one required computed property, description, which returns a String representation of the instance. If you adopt the protocol, you control exactly how your custom types are represented as printable String values.

  - First, adopt the protocol by adding a colon and the name of the protocol you want to adopt:

    - ```swift
        class Shoe: CustomStringConvertible {
          let color: String
          let size: Int
          let hasLaces: Bool
        }
      ```

  - Second, conform to the protocol by adding the required functions or variables. In this example, you'll add the description computed property. If you don't complete this step, the compiler will display an error that says you're not conforming to the protocol.
  - In the Strings lesson, you learned how to use string interpolation to interweave constants or variables into a String value. The description property in the following code uses string interpolation to create a String that includes each property of the Shoe instance.

    - ```swift
        class Shoe: CustomStringConvertible {
            let color: String
            let size: Int
            let hasLaces: Bool
         
            init(color: String, size: Int, hasLaces: Bool) {
                self.color = color
                self.size = size
                self.hasLaces = hasLaces
            }
         
            var description: String {
                return "Shoe(color: \(color), size: \(size), hasLaces: \(hasLaces))"
            }
        }
        let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
        let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
        print(myShoe)
        print(yourShoe)

        Console Output:
        Shoe(color: Black, size: 12, hasLaces: true)
        Shoe(color: Red, size: 8, hasLaces: false)
      ```

  - Note that this code prints out what looks like an initializer for each object. That's a common practice, because it enables you to see each property on the object in a familiar, concise format. However, you can print anything you want:

    - ```swift
        var description: String {
            let doesOrDoesNot = hasLaces ? "does" : "does not"
            return "This shoe is \(color), size \(size), and \(doesOrDoesNot) have laces."
        }
         
        let myShoe = Shoe(color: "Black", size: 12, hasLaces: true)
        let yourShoe = Shoe(color: "Red", size: 8, hasLaces: false)
        print(myShoe)
        print(yourShoe)

        Console Output:
        This shoe is Black, size 12, and does have laces.
        This shoe is Red, size 8, and does not have laces.
      ```

#### Comparing Information with `Equatable`

- Imagine your team is building an employee directory app for your company. You're tasked with building the data model to represent all the employees, their job titles, and their phone numbers.

  - ```swift
      struct Employee {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
      }
       
      struct Company {
        var name: String
        var employees: [Employee]
      }
    ```

- While building the app, you're asked to build a feature that allows any employee to edit their own information and prevents them from editing anyone else's information. You'll accomplish this by displaying an Edit button when the user navigates to their own detail screen.
- But how can you make sure it's the right employee? Each time the current employee navigates to an employee detail screen, you'll need to compare that employee to the employee whose information is displayed.
- Imagine you have access to the current employee with a `Session.currentEmployee` variable, which returns an Employee instance matching the current user. The view controller in charge of displaying the employee detail screen has an employee property. When the app loads a new employee detail screen, you'll need to check whether Session.currentEmployee and employee are equal. If so, you'll enable the Edit button. If not, you'll hide the button.
- You might consider writing something like the following:

  - ```swift
      let currentEmployee = Session.currentEmployee
      // Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Product Manager", phoneNumber: "415-555-0692")

      let selectedEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Marketing Director", phoneNumber: "415-555-9293")
       
      if currentEmployee == selectedEmployee {
        // Enable "Edit" button
      }
    ```

- The code above is a good thought. The == operator checks whether two values are equal. But because Employee is a custom type, you must tell Swift exactly how to compare two instances for equality. You'll do this by adopting the Equatable protocol.
- The Equatable protocol requires you to provide an implementation for the == operator on your custom type with a static == function that takes lhs (left-hand side) and rhs (right-hand side) parameters and returns a Bool that says whether the two values are equal:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
          // Logic that determines whether the value on the left-hand side and right-hand side are equal
        }
       
      }
    ```

- Consider the example above with two employees. How might you determine whether the two employees are equal? The following code returns true if both the firstName and lastName values are the same for both employees:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
          return lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName
        }
      }
    ```

- That wasn't a bad start at implementing an equality check. But it breaks down quickly. Consider two different employees with the same name:

  - ```swift
      let currentEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Industrial Designer", phoneNumber: "415-555-7766")
      let selectedEmployee = Employee(firstName: "James", lastName: "Kittel", jobTitle: "Marketing Director", phoneNumber: "415-555-9293")
       
      if currentEmployee == selectedEmployee {
        // Enable "Edit" button
      }
    ```

- This code returns true, so the employees would be able to edit each other's information in the directory.
- How can you design a better check to see whether two Employee instances are equal? You can update the == method to check for each property:

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func ==(lhs: Employee, rhs: Employee) -> Bool {
            return lhs.firstName == rhs.firstName && lhs.lastName == rhs.lastName && lhs.jobTitle == rhs.jobTitle && lhs.phoneNumber == rhs.phoneNumber
        }
      }
    ```

- If you compare the first names, last names, job titles, and phone numbers, you'll get a more reliable result. As you grow as a programmer, you'll start to recognize these types of edge cases, and you'll learn how to account for them in your code.
- Sometimes, the requirements of a protocol can be autogenerated for you. For example, if you specify that one of your types adopts Equatable and don't write your own == method, the Swift compiler will autogenerate an implementation that compares all the instance properties, like the one you just saw. So if all your == method is going to do is compare all the instance properties, you don't have to write it. The compiler will take care of it for you.
- To use autogeneration, your type must be a `struct` or `enum` — classes do not support autogeneration. Also, each property's type must also conform to Equatable.

  - ```swift
      struct Employee: Equatable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
      }
    ```

#### Sorting Information with `Comparable`

- Now imagine you're tasked with displaying a list of all the employees, sorted alphabetically by last name. Your manager gives you a sample list of five employees so you can experiment with writing code that will sort the list in the correct order.
- You start out with the following:

  - ```swift
      let employee1 = Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
      let employee2 = Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
      let employee3 = Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      let employee4 = Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      let employee5 = Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
       
      let employees = [employee1, employee2, employee3, employee4, employee5]
    ```

- Note that the list above isn't in any particular order.
- Swift provides a protocol named `Comparable`, that allows you to define how to sort objects using the <, <=, >, or >= operators.
- `Comparable` has two requirements: It requires that the type also adopt the Equatable protocol, and it requires the type to implement the < operator, which returns a Bool for whether the left-hand value is less than the right-hand value.
- In this case, you want to sort the employees alphabetically by last name. The String type is itself Comparable and uses the < operator to sort strings alphabetically, so you can implement the < function on Employee to return true if the last name of the left-hand value comes before the last name of the right-hand value alphabetically:

  - ```swift
      struct Employee: Equatable, Comparable {
        var firstName: String
        var lastName: String
        var jobTitle: String
        var phoneNumber: String
       
        static func < (lhs: Employee, rhs: Employee) -> Bool {
            return lhs.lastName < rhs.lastName
        }
      }
    ```

- You can then use the `sorted(by:)` function to return the array of employees sorted by last name:

  - ```swift
      let employees = [employee1, employee2, employee3, employee4, employee5]
       
      let sortedEmployees = employees.sorted(by: <)
       
      for employee in sortedEmployees {
        print(employee)
      }

      Console Output:
      Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
      Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
      Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")


      let sortedEmployees2 = employees.sorted(by:>)
       
      for employee in sortedEmployees2 {
        print(employee)
      }
      Console Output:
      Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
      Employee(firstName: "Glenn", lastName: "Parker", jobTitle: "Senior Manager", phoneNumber: "415-555-7770")
      Employee(firstName: "Stella", lastName: "Lee", jobTitle: "Accountant", phoneNumber: "415-555-7771")
      Employee(firstName: "Daren", lastName: "Estrada", jobTitle: "Sales Lead", phoneNumber: "415-555-7772")
      Employee(firstName: "Vera", lastName: "Carr", jobTitle: "CEO", phoneNumber: "415-555-7768")
    ```

- Swift is smart. It can use both the == operator and the < operator that you defined for the Equatable and Comparable protocols to provide functionality for the !=, <=, >, and >= operators as well.

#### Encoding and Decoding Objects with `Codable`

- Many apps save user input so that the data still exists the next time the user opens the app. To save user data, the values that live in memory must to be encoded to a form of data that can be written to a file. The Codable protocol makes this simple by creating key/value pairs from your object's property names and values that can then be used by an Encoder or Decoder object.
- Most Swift types that you use from the standard library already conform to Codable. If all of your custom type's properties conform to Codable, then all you have to do is add Codable to the type declaration, and the Swift compiler will autogenerate the necessary implementation.

  - ```swift
      struct Employee: Equatable, Comparable, Codable {
          var firstName: String
          var lastName: String
          var jobTitle: String
          var phoneNumber: String
       
          static func < (lhs: Employee, rhs: Employee) -> Bool {
              return lhs.lastName < rhs.lastName
          }
      }
    ```

- This lets an Encoder or Decoder object know that your type has all of the information it needs to be able to encode your object to or decode it from a certain data format.
- Take `JSONEncoder` as an example. The JSON data format, commonly used when working with web services, is essentially a list of key/value pairs that represent information. A JSONEncoder can convert an object conforming to Codable to JSON, which can then easily be encoded as Data or displayed as a String showing the list of key/value pairs.
- The `encode(_:)` method on JSONEncoder is a throwing function — a special type of Swift function that can return specific types of errors. You'll see the `try?` syntax, which allows the function to return an optional value, in the following sample code. If there's no error, the optional will hold the expected value; if there is an error, the optional will be nil. You'll learn more about try? in another lesson. If you get an error saying "Use of unresolved identifier JSONEncoder," make sure you import `Foundation`, the framework in which JSONEncoder is defined.

  - ```swift
      import Foundation
       
      let ben = Employee(firstName: "Ben", lastName: "Stott", jobTitle: "Front Desk", phoneNumber: "415-555-7767")
       
      let jsonEncoder = JSONEncoder()
      if let jsonData = try? jsonEncoder.encode(ben),
          let jsonString = String(data: jsonData, encoding: .utf8) {
          print(jsonString)
      }
       
      {"firstName":"Ben","lastName":"Stott","jobTitle":"Front Desk","phoneNumber":"415-555-7767"}
    ```

- The string printed to the console gives you a glimpse of how your object was converted into a different format but still represents the same information. By using Codable, you can easily convert your app's information to and from a variety of formats. The Codable protocol will come in handy in future lessons when you are saving user data or working with web services.

#### Creating a Protocol

- You've learned about four protocols defined in the Swift standard library, but you can also write your own protocols.
- Remember how to define structures, classes, and enumerations? You define a protocol in a very similar way. Use the protocol keyword followed by the name you want to use, and then define the requirements in a set of curly braces.
- When requiring a property, you must define whether the property is read-only or read/write. Read-only means you can get the variable, but you can't set it. Read/write means you can both get and set the value. If a property is read-only, you can implement it using a computed property. If it's read/write, it should be a regular property.
- When requiring a method, you need to specify the name, parameters, and return type of the method.
- The following code defines a FullyNamed protocol that requires a fullName property and a sayFullName() method. You can see that it looks similar to a type definition:

  - ```swift
      protocol FullyNamed {
        var fullName: String { get }
        // settable property
        // var fullName: String { get set }

        func sayFullName()
      }
    ```

- Just like with the Swift protocols earlier in this lesson, your types can then adopt the protocol by adding a colon and appending the name of the protocol.

  - ```swift
      struct Person: FullyNamed {
        var firstName: String
        var lastName: String
      }
    ```

- The compiler recognizes that the Person struct has adopted the protocol, but also recognizes that the struct doesn't yet meet the protocol's requirements. If you adopt a protocol but don't meet its requirements, you won't be able to build or run your code until you address the error.
- Notice how simple it is to conform to the protocol in the following code. The Person struct is updated with a fullyNamed computed property and a sayFullName() function that prints the full name to the console.

  - ```swift
      struct Person: FullyNamed {
        var firstName: String
        var lastName: String
       
        var fullName: String {
          return "\(firstName) \(lastName)"
        }
       
        func sayFullName() {
          print(fullName)
        }
      }
    ```

- You may have noticed that the syntax for adopting a protocol is similar to the syntax for declaring a subclass. That's not coincidental. Protocols and class inheritance are two ways to adopt a shared set of properties and functionality. In fact, you can use a protocol to provide a default implementation, just as a class can inherit an implementation from a superclass. You'll learn more about protocols and default implementations as you learn more about Swift.

#### Delegation

- Delegation is a design pattern, or common practice, that enables a class or structure to hand off, or delegate, some of its responsibilities to an instance of another type.
- It can help to think about delegation in the real world. If you delegate a task, you assign it to another person and expect them to handle it. For example, a manager may delegate a presentation to one of her employees, who would then be expected to prepare and deliver the presentation. Or a parent may delegate vacuuming the floor to one of their children, who would then handle the chore.
- The important thing to understand is that there are two sides: the one who's delegating the task, and the delegate whose job it is to actually do the task.
- This relationship is the same in programming. You might have an object that needs a task done, but it might not be the object that actually implements the code to make it happen. Types that delegate implementation to other types typically do so by defining a protocol. The protocol defines the responsibilities that can be delegated, and the delegate adopts the protocol to execute the actual task.
- The delegate pattern is especially important for working with frameworks which have objects that define built-in functionality but require you to provide behavior specific to your app. UIKit and other iOS frameworks use delegation extensively. They provide objects such as `UIApplication`, `UITextField`, and `UITableView`, which use delegation to customize their behavior. For example, a `UIApplicationDelegate` can define how an app responds to remote notifications, a `UITextFieldDelegate` can validate text input, and a `UITableViewDelegate` can perform actions when a row is tapped.
- You'll work more with the delegate pattern in future lessons.
- Protocols are one of the more advanced Swift topics you'll use in this course. Don't be intimidated. You'll learn to master protocols as you use them while working your way through the course.

#### Lab

- App Exercise - Heart Rate Delegate
- These exercises reinforce Swift concepts in the context of a fitness tracking app.
- Your fitness tracking app will likely have a class dedicated to receiving information from the fitness tracking hardware. The HeartRateReceiver class below represents a very simplified example of what this may look like with monitoring heart rate. The function startHeartRateMonitoringExample will generate random heart rates and assign them to currentHR, simulating how an instance of HeartRateReceiver may pick up on new heart rate readings at specific intervals.
- HeartRateViewController below is a view controller that will present the heart rate information to the user. Throughout the exercises below you'll use the delegate pattern to pass information from an instance of HeartRateReceiver to the view controller so that anytime new information is obtained it is presented to the user.

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }

          func startHeartRateMonitoringExample() {
              for _ in 1...10 {
                  let randomHR = 60 + Int(arc4random_uniform(UInt32(15)))
                  currentHR = randomHR
                  Thread.sleep(forTimeInterval: 2)
              }
          }
      }

      class HeartRateViewController: UIViewController {
          var heartRateLabel: UILabel = UILabel()
      }
    ```

- First, create an instance of HeartRateReceiver and call startHeartRateMonitoringExample. Notice that every two seconds currentHR get set and prints the new heart rate reading to the console.

  - ```swift
      let receiver1 = HeartRateReceiver()
      receiver1.startHeartRateMonitoringExample()

      console:
      The most recent heart rate reading is 66.
      The most recent heart rate reading is 72.
      The most recent heart rate reading is 68.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 71.
      The most recent heart rate reading is 71.
      The most recent heart rate reading is 60.
      The most recent heart rate reading is 70.
      The most recent heart rate reading is 62.
    ```

- In a real app, printing to the console does not show information to the user. You need a way of passing information from the `HeartRateReceiver` to the `HeartRateViewController`. To do this, create a protocol called `HeartRateReceiverDelegate` that requires a method `heartRateUpdated(to bpm:)` where `bpm` is of type `Int` and represents the new rate as _beats per minute_. Since playgrounds read from top to bottom and the two previously declared classes will need to use this protocol, you'll need to declare this protocol above the declaration of `HeartRateReceiver`.

  - ```swift
      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }
    ```

- Now make `HeartRateViewController` adopt the protocol you've just created. Inside the body of the required method you should set the text of `heartRateLabel` and print "The user has been shown a heart rate of <INSERT HEART RATE HERE>."

  - ```swift
      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }
      class HeartRateViewController: UIViewController, HeartRateReceiverDelegate {
          var heartRateLabel: UILabel = UILabel()

          func heartRateUpdated(to bpm: Int) {
              heartRateLabel.text = "The user has been shown a heart rate of \(bpm)"
              print("The user has been shown a heart rate of \(bpm)")
          }
      }
    ```

- Now add a property called `delegate` to `HeartRateReceiver` that is of type `HeartRateReceiverDelegate?`. In the `didSet` of `currentHR` where `currentHR` is successfully unwrapped, call `heartRateUpdated(to bpm:)` on the `delegate` property.

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                      delegate?.heartRateUpdated(to: currentHR)
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }
          var delegate: HeartRateReceiverDelegate?

          ...
      }
    ```

- Finally, return to the line of code just after you initialized an instance of `HeartRateReceiver`. Initialize an instance of `HeartRateViewController`. Then, set the `delegate` property of your instance of `HeartRateReceiver` to be the instance of `HeartRateViewController` that you just created. Wait for your code to compile and observe what is printed to the console. Every time that `currentHR` gets set, you should see both a printout of the most recent heart rate, and the print statement stating that the heart rate was shown to the user.

  - ```swift
      let receiver2 = HeartRateReceiver()
      let hrController = HeartRateViewController()
      receiver2.delegate = hrController
      receiver2.startHeartRateMonitoringExample()

      Console:
      The most recent heart rate reading is 61.
      The user has been shown a heart rate of 61
      The most recent heart rate reading is 61.
      The user has been shown a heart rate of 61
      The most recent heart rate reading is 68.
      The user has been shown a heart rate of 68
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 63.
      The user has been shown a heart rate of 63
      The most recent heart rate reading is 64.
      The user has been shown a heart rate of 64
      The most recent heart rate reading is 67.
      The user has been shown a heart rate of 67
      The most recent heart rate reading is 62.
      The user has been shown a heart rate of 62
      The most recent heart rate reading is 70.
      The user has been shown a heart rate of 70
    ```

- The final

  - ```swift
      class HeartRateReceiver {
          var currentHR: Int? {
              didSet {
                  if let currentHR = currentHR {
                      print("The most recent heart rate reading is \(currentHR).")
                      delegate?.heartRateUpdated(to: currentHR)
                  } else {
                      print("Looks like we can't pick up a heart rate.")
                  }
              }
          }

          var delegate: HeartRateReceiverDelegate?

          func startHeartRateMonitoringExample() {
              for _ in 1...10 {
                  let randomHR = 60 + Int(arc4random_uniform(UInt32(15)))
                  currentHR = randomHR
                  Thread.sleep(forTimeInterval: 2)
              }
          }
      }

      class HeartRateViewController: UIViewController, HeartRateReceiverDelegate {
          var heartRateLabel: UILabel = UILabel()

          func heartRateUpdated(to bpm: Int) {
              heartRateLabel.text = "The user has been shown a heart rate of \(bpm)"
              print("The user has been shown a heart rate of \(bpm)")
          }
      }
      //let receiver1 = HeartRateReceiver()
      //receiver1.startHeartRateMonitoringExample()

      protocol HeartRateReceiverDelegate {
          func heartRateUpdated(to bpm: Int)
      }

      let receiver2 = HeartRateReceiver()
      let hrController = HeartRateViewController()
      receiver2.delegate = hrController
      receiver2.startHeartRateMonitoringExample()
    ```

### Lesson 1.2 App Anatomy and Life Cycle

- In this lesson, you'll learn more about the different life cycle states and the delegate hooks for executing logic as the app moves through each state.
- Most iOS apps run when the user opens them, and they stop running when the user returns to the Home screen, uses the app switcher, or otherwise switches to a different app.
- As a programmer, you have delegate methods, or hooks where you can execute code, at any of these events in your app's life cycle. For example, when the app launches, you may want to trigger a network call to fetch new data. When the app closes, you may want to save your user's progress. App and scene delegates also ensure your app works properly with iOS, multiple instances of your app, and with other iOS apps.
- In this lesson, you'll examine the AppDelegate.swift and SceneDelegate.swift files that Xcode creates with every new project. This will help you learn the most common delegate methods that are called as the app transitions through its life cycle, such as when it moves from the foreground to the background.
- But before digging into the code, you should understand the different stages of the app life cycle.
- Before digging into the code, you should understand the different stages of the app life cycle.

  | App State | Description |
  | --------- | ----------- |
  | Not running | The app has not been launched or has been terminated, either by the user or by the system. |
  | Inactive | The app is running in the foreground but isn't receiving touch events. (It may be executing other code, though.) An app usually stays inactive only briefly as it transitions to a different state. |
  | Active | The app is running in the foreground and receiving events. In the active state, an app has no special restrictions placed on it and should be responsive to the user. |
  | Background | The app is executing code but is not visible onscreen. When the user quits an app, the system moves the app to the background state briefly before suspending it. At other times, the system may launch the app into the background (or wake up a suspended app) and give it time to handle specific tasks. For example, the systemm may wake an app so that it can process background downloads, certain types of location events, remote notifications, and other events. An app in the background state should do as little work as possible. |
  | Suspended | The app is in memory but isn't executing code. The system will suspend apps that are in the background may purge suspended apps at any time, without waking them up, to make room for other apps. |

  - <img src="./resources/app_delegate.png" alt="App Delegate" width="500" />

#### Break Down the App Delegate

- Create a new project using the iOS App template and name it "AppLifeCycle." When creating the project, make sure the interface option is set to "Storyboard."
- In the Project navigator, open AppDelegate, which defines the AppDelegate class. The AppDelegate class conforms to the UIApplicationDelegate protocol, which defines methods that serve as hooks into the different events in the app's life cycle.
- Take a look at the methods included in the app delegate. Read through the comments to learn about what the methods do. Prior to iOS 13, the app delegate had a much larger role in the app life cycle. iOS 13 introduced the ability for apps to have multiple instances running on iPadOS, each of which is managed by a UISceneDelegate.
- While it's possible to forgo using UISceneDelegate, falling back to using UIApplicationDelegate as it was prior to iOS 13, it's best to conform to any new design patterns that have been introduced. By doing so you'll be in a better position to handle any future features, devices, or deprecations — or simply to extend your app to support multiple scenes on iPad when you're ready. In this course, you'll adhere to scene - based life cycle semantics, but all your apps will only have a single scene — you won't do any work to explicitly support multiple scenes.

- **Did Finish Launching**

  - When your app has finished launching, the first method will be called, providing your first opportunity to run your own code in the app. This capability isn't duplicated in a scene delegate, so you'll need to use your app delegate to perform actions when your app is first launched.

    - ```swift
        func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
            // Override point for customization after application launch.

            return true
        }
      ```

  - The app delegate also manages scenes as they're connected and disconnected.

- **Configuration For Connecting a Scene Session**

  - The next function is called when a new scene session is being created. By default Xcode generates your project with an Info.plist that has the information necessary to create a default scene configuration, named “Default Configuration”, as evident in this implementation. This all you need to get started – supporting various scene configurations is an advanced topic that won't be covered here.

    - ```swift
        func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) ->
          UISceneConfiguration {
            // Called when a new scene session is being created.
            // Use this method to select a configuration to create the new scene with.

            return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
        }
      ```

- **Did Discard Scene Sessions**

  - When a user discards an instance of your app on iPad, this method is called with information about that scene. It's possible you'll receive multiple scene sessions if the user discards multiple instances at the same time or if the action was taken while your app was suspended. It's up to you to decide what should happen in your app when this event occurs – keeping in mind what your users might reasonably expect to happen.

    - ```swift
        func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
            // Called when the user discards a scene session.
            // If any sessions were discarded while the application was not running, this will be called shortly after didFinishLaunchingWithOptions.
            // Use this method to release any resources that were specific to the discarded scenes, as they will not return.
        }
      ```

  - There are other capabilities unique to the app delegate, but most basic lifecycle events can be handled by a scene delegate.

#### Break Down the Scene Delegate

- In the Project navigator, open SceneDelegate. Your scene delegate is responsible for handling UI-level components during your app's life cycle. These methods will be called for all scenes that your app has created. You'll notice that each method receives a UIScene instance — this is the scene that the event occurred for.

  - <img src="./resources/scene_delegate.png" alt="Scene Delegate" width="500" />

- Read through the provided methods and their comments to learn about what the methods do. As always, you can Option-click a method's signature to see more information.

- **Will Connect To**

  - This method is called with a scene instance that is being connected to your app. This provides you with an opportunity to perform any necessary steps to prepare for the additional scene such as fetching data from your local persistence store or opening network connections.

    - ```swift
        func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
            // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
            // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
            // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).

            guard let _ = (scene as? UIWindowScene) else { return }
        }
      ```

- **Scene Did Disconnect**

  - This method, called when the scene has been removed from your app, is the best place to clean up and release any resources or to save files that were necessary for the scene. UIKit disconnects scenes when a user explicitly closes them in the app switcher and can disconnect scenes in the background state on behalf of the system to free up resources for other active applications. It's not as commonly overridden as sceneDidEnterBackground(_:) below. A scene is not guaranteed to be disconnected when the user switches to another app.

    - ```swift
        func sceneDidDisconnect(_ scene: UIScene) {
            // Called as the scene is being released by the system.
            // This occurs shortly after the scene enters the background, or when its session is discarded.
            // Release any resources associated with this scene that can be re-created the next time the scene connects.
            // The scene may re-connect later, as its session was not necessarily discarded (see `application:didDiscardSceneSessions` instead).
        }
      ```

- **Scene Did Become Active**

  - This method is called to let your scene know that it moved from the foreground inactive state to the foreground active state, which can occur when a user switches to your scene. Scenes can also return to the foreground active state if the user chooses to ignore an interruption, such as an incoming phone call or system alert, that temporarily sent the app into the inactive state.

    - ```swift
        func sceneDidBecomeActive(_ scene: UIScene) {
            // Called when the scene has moved from an inactive state to an active state.
            // Use this method to restart any tasks that were paused (or not yet started) when the scene was inactive.
        }
      ```

- **Scene Will Resign Active**

  - The next method is called when the scene is about to leave the foreground active state. This event can happen when the user has closed the scene or quit the app, but it's also called when the app is interrupted temporarily by a phone call or system alert.

    - ```swift
        func sceneWillResignActive(_ scene: UIScene) {
            // Called when the scene will move from an active state to an inactive state.
            // This may occur due to temporary interruptions (ex. an incoming phone call).
        }
      ```

- **Scene Will Enter Foreground**

  - This method is called as part of the transition from the background state to the foreground active state — immediately before sceneDidBecomeActive(_:). You can use this method to undo many of the changes made to your scene when it entered the background.

    - ```swift
        func sceneWillEnterForeground(_ scene: UIScene) {
            // Called as the scene transitions from the background to the foreground.
            // Use this method to undo the changes made on entering the background.
        }
      ```

- **Scene Did Enter Background**

  - The next method is called immediately after the sceneWillResignActive(_:) method, once the scene has actually entered the background state. This is the right time to spin down demanding processes and save the user's work or progress.

    - ```swift
        func sceneDidEnterBackground(_ scene: UIScene) {
            // Called as the scene transitions from the foreground to the background.
            // Use this method to save data, release shared resources, and store enough scene - specific state information to restore the scene back to its current state.
        }
      ```

#### Try It Out

- Add a print statement to each of the delegate methods in both AppDelegate and SceneDelegate that describes what happens to trigger the method. For example, print("Application did finish launching.") in the application`(_:didFinishLaunchingWithOptions:)` method, and print("Scene will resign active.") in `sceneWillResignActive(_:)`.
- Run the app in Simulator. When you open the app, you'll see four messages print to the console:
  - Application did finish launching.
  - Scene will connect to session.
  - Scene will enter foreground.
  - Scene did become active.
- Dismiss the app and return to the Home screen. You'll see two more messages print to the console:
  - Scene will resign active.
  - Scene did enter background.
- Open the app switcher and reopen the app. You'll see another two messages print to the console:
  - Scene will enter foreground.
  - Scene did become active.
- Open the app switcher again, then return to the app. You'll see two more messages print to the console:
  - Scene will resign active.
  - Scene did become active.
- Take note of the various delegate methods and how you might use them when building your apps. As you can see from this exercise, they provide useful hooks to execute code at each app transition—even though you may not need to use all of them in every app you build.

#### Which Method Should I Use?

- You've learned about the many options for responding to different transitions in your app. For now, focus on three methods that will run when launching, reopening, or closing your app:
  - `application(_:didFinishLaunchingWithOptions:)`
  - `sceneDidBecomeActive(_:)`
  - `sceneWillResignActive(_:)`
- As you become more experienced and build more complex apps, you'll encounter situations where you'll want to take advantage of the other delegate methods.

#### Lab - App Event Count

- **Objective**

  - The objective of this lab is to create an app that provides a visual representation of the app life cycle. Your app will update labels on the view when different delegate methods are called.
  - Create a new project called "AppEventCount" using the iOS App template. When creating the project, make sure the interface option is set to "Storyboard."

- **Step 1 Add Event Counters to AppDelegate**

  - Open AppDelegate and create two variables at the top to count the number of times the app has launched and the number of times it has created a configuration for connecting to a scene.

    - ```swift
        var launchCount = 0
        var configurationForConnectingCount = 0​
      ```

  - Increment both of these counts by 1 within the corresponding methods.

- **Step 2 Set Up Your View and View Controller**

  - As with all apps created using the iOS App template, your app starts out with one view controller. Drag seven labels to it, one for each of the following seven AppDelegate and SceneDelegate life cycle methods. Set up constraints as necessary. Hint: Consider using a stack view.
    - application(_:didFinishLaunchingWithOptions:)
    - application(_:configurationForConnecting:options:)
    - scene(_:willConnectTo:options:)
    - sceneDidBecomeActive(_:)
    - sceneWillResignActive(_:)
    - sceneWillEnterForeground(_:)
    - sceneDidEnterBackground(_:)
  - You'll use the labels to display the number of times that each event has occurred. Create an outlet for each label, using descriptive names like `didFinishLaunchingLabel` and `didBecomeActiveLabel`.
  - Also, for each label corresponding to the scene events (not AppDelegate events), create a variable to store a count of how many times its delegate method has been called. Set the initial value of each variable to 0, as in the following example:
`var willConnectCount = 0`

- **Step 3 Access Count vars From AppDelegate**

  - To update your UI for the application(_:didFinishLaunchingWithOptions:) and application(_:configurationForConnecting:options:) calls, you'll need access to the two variables you created in AppDelegate. In ViewController, add the following variable: `var appDelegate = (UIApplication.shared.delegate as! AppDelegate)`
  - This will allow you to access AppDelegate and your count variables within ViewController. Your AppDelegate instance is known as a singleton and can be accessed in this manner. Since you know what the type is, you can safely downcast it using `as! AppDelegate`.
  - ​​While it may be convenient to access AppDelegate anywhere, making it tempting to fill it with information, it's a best practice to keep it focused on its responsibilities. In this case, you're accessing it for information that it is responsible for.
  - Create an updateView() method that updates each label with its count. For the AppDelegate events, you'll access your variables via your new appDelegate variable, and for scene events, you'll use the instance variables that you created within ViewController. Update the labels in updateView(), as in the following example: `launchLabel.text = "The App has launched \(appDelegate.launchCount) time(s)"`

- **Step 4 Give SceneDelegate Access To ViewController**

  - In the SceneDelegate class, just below the line var window: UIWindow?, create a variable property called `viewController` that is of type `ViewController?`. Be sure to make it optional.
  - At the top of the scene(_:willConnectTo:options:) method, set the property `viewController` equal to the `rootViewController`. This will give the SceneDelegate access to the instance of ViewController, enabling you to write code that increments the corresponding count properties on ViewController each time an app life cycle method has been called, as in the following example: `viewController = window?.rootViewController as? ViewController`
 
- **Step 5 Increment the Count Properties**

  - In scene(_:willConnectTo:options:), increment the count of the ViewController property corresponding to the scene connecting: `viewController?.willConnectCount += 1`
  - Repeat this step for the other SceneDelegate methods:
    - sceneDidBecomeActive(_:)
    - sceneWillResignActive(_:)
    - sceneWillEnterForeground(_:)
    - sceneDidEnterBackground(_:)

- **Step 6 Regularly Update the View**

  - Your view will need to update regularly to display the new counts for each app event. You might think to call updateView() in the viewDidLoad() method of your ViewController, but there's a problem: viewDidLoad() won't be called at all the app events. 
  - There's a better place to update the view. One of the scene event methods, `sceneDidBecomeActive(_:)`, will be called after each of the other life cycle methods is called and just before the user can interact with the scene again. The `sceneDidBecomeActive(_:)` method is the perfect place to call the view controller's updateView(), ensuring that the labels display the proper count.

- **Step 7 Test**

  - Run the app in Simulator.
  - The labels for both the app did finish launching and the scene did become active events should show 1 for their counts.
  - Go to the Home screen on Simulator.
  - Return to the app and observe which other labels have changed.
  - Bring up the app switcher, but instead of changing apps, return to the same app. Which labels have changed?
  - When the user quits your app or dismisses a scene, `sceneDidDisconnect(_:)` will be called—but adding a label and counter the same way for this method would be ineffective, as the data would be lost between terminations.
  - Great job! You've made an app that helps you visualize the app life cycle. Be sure to save it to your project folder.

### Lesson 1.3 Model-View-Controller

#### Overview

- At this stage of the course, you're probably beginning to feel more comfortable with how apps work and how to build basic app features. You've seen how even relatively simple apps rely on many files, structures, and classes. Imagine how the code for a medium-sized app might span across hundreds of files in your project.
In this lesson, you'll learn how to organize files, structures, and classes into a design pattern called Model-View-Controller, or MVC. MVC will help you architect the files in your app as well as the interactions and relationships between different types and instances.
- MVC is not a trivial topic. This lesson will help you get started, but you'll continue learning about and reinforcing your understanding of MVC concepts for a long time. You'll learn the intricacies of implementing proper MVC patterns in your app from sample code, from building your own apps, and from mentors and teachers.
- Keep in mind that there's never one right answer to any architecture decision. There are best practices, but style and personal preference will play a role in how you choose to organize your app's projects and relationship models.
- Now that you've started to build more complex projects, you know quite a bit about how different classes and structures work together. And you've probably noticed some patterns emerge.
- For example, you know you can use AppDelegate and SceneDelegate methods to handle different events in the app life cycle. You know you can edit the Main storyboard file in Interface Builder to define views and create user interfaces. You've worked with view controller files that define how a scene should respond when the user taps a button or navigates between screens. And you've seen specific classes that represent data for your app to display.
- So far in this course, you've followed specific instructions that told you what classes or structures to create and which properties or functions to add to them. But before that, someone had to decide how all the pieces were going to work together. How would you decide which new classes or structures to create? How do you know what properties to have? Or which objects should call functions on other objects?
- These are all great questions. And by asking them, you're graduating from someone who's simply following instructions to a programmer who's making decisions.
- For decades, software developers have studied and practiced different ways of answering these questions. As software development has matured, a few common design patterns have emerged. One of the most prevalent patterns is Model-View-Controller, or MVC. MVC assigns objects to one of three roles — model, view, or controller — and helps define the way objects communicate with each other.
- The diagram below provides an overview of how the three types of objects work in relation to one another. Each layer, or type, has specific roles and guidelines for communicating with the other layers.

  - <img src="./resources/overview_mvc.png" alt="Overview MVC" width="400" />

- Before trying to understand how the interactions work, you should understand what each type of object is responsible for. As you read the descriptions below, revisit the diagram and review the interactions between the types of objects.

#### Object - Model

- A model object groups the data you need to represent items or concepts. These items or concepts might be unique to your app, such as a character in a game or a task in a to-do list, or they might represent things in the real world, such as a person and their contact information in an address book, or an item to be purchased from a store.
- Model objects can be related to other model objects. A model object for a car might have a driver property, or that game character might have an array of inventory items.
- In most cases, you'll create model objects by defining structures or classes in your project. And you'll typically define the structures or classes in new Swift files. 
- Model objects are made up of properties that represent attributes of the type, and they sometimes have methods for updating and modifying their own properties.
- **Communication**
  - Model objects are usually created in response to some user interaction with a view or a control. The message to create the model object is transmitted from the view through a controller object, which is most often a subclass of a view controller. That said, model objects

#### Object - View

- You've already learned that views are the visual aspects of the user interface. A view object knows how to draw itself on the screen and can respond to user input. A major purpose of view objects is to display data about an app's model objects and to allow the user to edit that data. 
- Views can be reused or reconfigured to show different instances of model data. For example, a view object in the Contacts app can be used to display information about any contact, and a view in Mail can be used to show any message.
- Views often have an update or configure function that accepts a model object as a parameter; the view uses the model object to update itself to match the data it's meant to display. For example, imagine a table view cell displaying a photo of a player along with the player's current score. The view might have an update(_:) method that assigns the player's image to an image view and the player's score to a label.
- When building an iOS app, you'll typically define the view layer in Interface Builder. Views can then be referenced or used in a view controller class as outlets or actions.
- **Communication**
  - Even though views are commonly used to display data about an app's model objects, views should never own a model object as a property or modify a model object directly. Instead, the view can send a message — for example, that the user has performed an action — to a view controller, and the controller will update the model object.

#### Object - Controller

- A controller object acts as the messenger between views and model objects. For example, when the user interacts with a view, the view sends a message to a view controller, and the view controller can then update the model object. Alternatively, when a model object is created or updated, a view controller can tell its view to redraw or update itself with the new data.
- In fact, the view controller is the most common type of controller for new iOS developers. As you've learned, a view controller controls a view along with its accompanying subviews and usually displays information about one or more model objects. The model object, or collection of model objects, is usually defined as a property on the view controller. When the user interacts with a view or control, an action or block of code is triggered on the view controller, which can then update the model object.
- There are two other types of controllers: model controllers and helper controllers.
- **Model Controllers**
  - Similar to how a view controller controls a view, a model controller helps control a model object or a collection of model objects. There are three common reasons you might want to create a model controller:
    - Multiple objects or scenes need access to the model data.
    - The logic for adding, modifying, or deleting model data is complex.
    - You want to keep the code in your view controllers focused on managing the views.
  - For example, imagine you're working on a simple Notes app that syncs the user's notes to a server. The app has two scenes: a list view, which displays all the notes in a table view, and a detail view, which allows the user to create a new note or read and edit a preexisting note.
  - If you used only view controllers, the list view controller would take on the majority of the work in the app — not only managing the list view but also managing all the model data, including fetching the note data from the server, adding new notes, replacing modified notes, deleting notes, and saving all changes to the server.
  - Instead, you might decide to abstract, or move, all the code that manages notes to a separate model controller called NoteController. This approach allows the view controller to focus only on displaying the notes and leaves the networking and model management to the model controller. 
  - Using model controllers isn't required; in fact, it's not a common practice in small projects. But abstraction is crucial when working on larger projects or working with other developers, because it makes your code simpler, more readable, and easier to maintain.
- **Helper Controllers**
  - Helper controllers are useful anytime you want to consolidate related data or functionality so that it can be accessed by other objects in your app. One common example of a helper controller is a NetworkController, which manages all the network requests in a given app.
  - Controllers should perform a very specific function without depending on other controllers to perform their work. For example, a NoteListViewController might display a list of notes, and a NoteDetailViewController might display the details about an individual note instance and respond to user input to modify or create a note.
- **Communication**
  - Besides mediating the communication between models and views, controller objects can communicate or work directly with other controller objects. Consider the examples above. The initial view controller of a Notes app (NoteListViewController) might be responsible for displaying a list of notes, so it accesses the notes property on a note model controller (NoteController). The note model controller needs to check whether there are any new notes, so it tells the network controller (NetworkController) to check the web service for new data. If the NetworkController finds new notes, it downloads the data and sends it back to the NoteController, which would then update its notes property and send a callback to the NoteListViewController that it has new data, enabling the NoteListViewController to update its view of notes.

#### Example

- Imagine you're tasked with building an app to track photos of meals that the user has eaten, along with notes and an optional rating of each meal. What model objects will you need? What views? What controllers?
- Before moving on, take a moment to consider how you might plan, or architect, this simple app using model, view, and controller objects.
- **Model**
  - If the app is going to track meals, it makes sense to have a model object called Meal that holds the data about each meal. Each meal should have a name, and — according to your feature description — it also needs to have a photo, notes, and a rating.
  - Next, you'll need to consider how your model objects will be displayed in the app — and your decisions will impact the properties to keep on your model objects. For example, if you want to make sure that a list of meals is displayed in the order that the meals were added, you may need to include a timestamp on each meal object.
  - So your model object might be declared as a structure or a class with five properties:

    - ```swift
        struct Meal {
            var name: String
            var photo: UIImage
            var notes: String
            var rating: Int
            let timestamp: Date
        }
      ```

- **View**
  - You can imagine that the app will need at least two views, or scenes. Just like in the Notes example, one scene will display a list of all the meals that the app has tracked, and another scene will display the details of a meal.
  - The meal list scene could display the list in a table view, with the name and photo of each meal in an individual cell. The meal detail scene could then display the name of the meal in the navigation bar, a photo in an image view, notes about the meal in a text view, and the current rating in a segmented control.
  - Each of these two views will need a view controller class that can manage the model data it's displaying and can respond to user interactions.
  - In most cases, you'll use a storyboard to define your views in Interface Builder. You'll assign each view a view controller class, where you can add actions or outlets to respond programmatically to user interaction and to update the view.
  - Occasionally, you may build a view that requires its own class definition. For example, you may have a custom table view cell with a button, which would require you to create a subclass of the cell type and add functionality in the class definition. You'll learn more about custom views with subclasses later.
- **Controller**
  - At a minimum, you'll need a controller for the meal list and meal detail views. For the meal list, you should use a UITableViewController, which displays data in a table, and for the detail view controller you should use a UIViewController. Some of the table view setup that follows will be unfamiliar to you, but don't worry — you'll learn more about UITableViewController and setting up table views in a future lesson.
- **Meal List Table View Controller**
  - `class MealListTableViewController: UITableViewController {...}`
  - The MealListTableViewController should have a property that holds a collection of meals to display.
  - The properties you add to a table view controller subclass — in this case, MealListTableViewController — should reflect anything that makes it different from a plain UITableViewController.
    - `class MealListTableViewController: UITableViewController { var meals: [Meal] = [] }`
  - Because the collection of meals lives on MealListTableViewController, the controller should handle adding and deleting meals from the collection. One approach would be to modify the array directly, but you might choose instead to define functions that add or remove meals from the array.
  - When the user quits the app, they'll expect it to save any meal data they just entered or modified. In this example, MealListTableViewController is the intermediary between the view and the meal model data, so it should take responsibility for saving data to disk, as well as for loading data when the app launches and before the view is displayed.

    - ```swift
        class MealListTableViewController: UITableViewController {
            var meals: [Meal] = []
            func saveMeals() {...} 
            func loadMeals() {...}
        }
      ```

  - The view controller should also respond to any user events. What events might the list view handle?
  - List views usually let the user select an item to display more details about the item. You'd enable that functionality by adding a segue from the list view scene to the detail view scene in the storyboard. You'll also need to add and implement the `prepare` function that passes the selected meal to the detail scene.
  - When all is said and done, you'll likely end up with a view controller that looks similar to this:

    - ```swift
        class MealListTableViewController: UITableViewController {
            var meals: [Meal] = []
            override func viewDidLoad() {
                super.viewDidLoad()
                // Load the meals and set up the table view
            }
            // Required table view methods

            override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {...}

            override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {...}

            // Navigation methods

            @IBSegueAction func showMealDetails(_ coder: NSCoder) -> MealDetailViewController? {
                // Initialize MealDetailViewController with selected Meal and return
            }

            @IBAction func unwindToMealList(sender: UIStoryboardSegue) {
                // Capture the new or updated meal from the MealDetailViewController and save it to the meals property
            }

            // Persistence methods

            func saveMeals() {
                // Save the meals model data to the disk
            }

            func loadMeals() {
                // Load meals data from the disk and assign it to the meals property
            }
        }
      ```

  - Once you've planned the required features of the list view controller, consider the detail view controller. How will the two controllers interact with each other?
- **Meal Detail View Controller**
  - `class MealDetailViewController: UIViewController {...}`
  - The MealDetailViewController will display the details about an individual meal. You can use Interface Builder to set up views for displaying the name, the image, the notes, and the rating, then create outlets to MealDetailViewController. Your view controller should also have a property for the meal that will be displayed.
  - Providing a custom initializer and using@IBSegueAction to display the detail controller from MealListTableViewController allows you to guarantee that a model is provided, so that you do not need to worry about handling optional values.

    - ```swift
        class MealDetailViewController: UIViewController {
            var meal: Meal

            @IBOutlet var nameTextField: UITextField!
            @IBOutlet var photoImageView: UIImageView!
            @IBOutlet var ratingControl: RatingControl!
            @IBOutlet var saveButton: UIBarButtonItem!

            init?(coder: NSCoder, meal: Meal) {
                self.meal = meal
                super.init(coder: coder)
            }

            override func viewDidLoad() {
                super.viewDidLoad()
                // Update view components using the Meal model
            }
        }
      ```

  - The MealDetailViewController should handle updating the view it controls with the information about the model object (the meal) it's meant to display. Many times you can do this right in viewDidLoad(), but you may want to break the process out into its own method — especially if the model can change.
  - The detail view controller might have additional methods to allow the user to add a photo or to modify other properties of the meal. In those scenarios, it will need some way to know whether it's displaying an existing meal or creating a new meal.
  - After accounting for all the tasks the detail view controller is handling, the result might look something like this:

    - ```swift
        class MealDetailViewController: UIViewController, UIImagePickerControllerDelegate {

            @IBOutlet var nameTextField: UITextField!
            @IBOutlet var photoImageView: UIImageView!
            @IBOutlet var ratingControl: RatingControl!
            @IBOutlet var saveButton: UIBarButtonItem!

            var meal: Meal

            init?(coder: NSCoder, meal: Meal) {
                self.meal = meal
                super.init(coder: coder)
            }

            override func viewDidLoad() {
                super.viewDidLoad()
                // Update view components using the Meal model
            }

            // Navigation methods

            override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
                // Update the meal property that will be accessed by the MealListTableViewController to update the list of meals
            }

            @IBAction func cancel(_ sender: UIBarButtonItem) {
                // Dismiss the view without saving the meal
            }
        }
      ```

- **A Reminder**
  - MVC is a tool to help you write clean, maintainable, and organized code. But there's more than one way to implement it. If you were to plan out the same classes for the same app, you might end up with different variable names, different function names, different model objects, and different interfaces for your view hierarchies.
  - When thinking through MVC, developers have their own distinct styles and patterns. Since you're going through this course, you'll tend to follow the patterns used here. But as you grow and gain more experience, you'll come up with your own preferences for how to architect the objects in your apps.
  - You'll also develop your own perspective on the advantages and guidelines behind the MVC design pattern. Look at sample code and previous projects for further guidance. And don't be afraid to ask someone with more experience to mentor you in the best approach to app architecture.

#### Project Organization

- Xcode projects get crowded quickly. Even small apps can have dozens or hundreds of files that define different views, storyboards, structures, classes, protocols, and controllers. Writing navigable code is an important part of building any program. If you aren't careful, your project can quickly become disorganized, which will make it hard to continue working productively.
- The first step in organizing your project is to create clear, descriptive filenames. For example, ProductListTableViewController.swift is much more descriptive than MainVC.swift. Descriptive names for classes, structures, protocols, properties, and functions will also help you stay productive. Some developers worry that they'll have to write long names over and over again, but the autocompletion feature in Xcode makes it easy, so there's no advantage to a short name.
- Another good practice is to create separate files for each of your type definitions, such as classes, structures, protocols, and enumerations. For example, create separate Car.swift, Driver.swift, RaceTrack.swift, and Garage.swift files, rather than one combined Model.swift file.
- One more thing: Write your code as if complete strangers are going to read it. You might think this only matters when you're working with other people, but it also applies to code only you will see. When you need to revise your code six months down the road, you'll appreciate descriptive names, clear function signatures, and details comments.
- Xcode also provides a grouping feature to help you organize code. It's basically a way to group your files into folders within your project, so that you don't end up with one never-ending list of files in the Project navigator.
- Many developers make groups for the following:
  - View controllers
  - Views
  - Models
  - Model controllers
  - Other controllers
  - Protocols
  - Extensions
  - Resources
    - Storyboards
    - Frameworks
- For now, you might want to keep your AppDelegate, SceneDelegate, and Main files on the top level and use a shorter list of groups:
  - Models
  - Other controllers
  - Storyboards
  - View controllers
  - Views

    - <img src="./resources/list_of_group.png" alt="List of Groups" width="200" />

- It's worth noting that the way you organize your project in Xcode may not always change the way your files are saved in the project directories; depending on how groups are created, it might only affect how files are displayed within Xcode itself. In Xcode, this is signified by two different folder icons. A group that is used to organize information logically in your project but that does not represent a folder that you would see in Finder is denoted by a shaded lower-left corner in the folder icon.

  - <img src="./resources/folder_icons.png" alt="Folder Icons" width="200" />

- When creating groups in Xcode's navigator, you will see two options: New Group and New Group without Folder. The latter does not create a folder on disk — it's simply visual organization within Xcode.
- The specifics of how to organize your projects will depend on what you're building. For now, think simple. Allow yourself time to keep things organized so that your projects are easy to navigate. You'll appreciate it later.

#### Lab - Favorite Athletes

- **Object**
  - In this lab, you'll plan out and create an app that uses proper MVC design. Your app will have two screens for displaying the user's favorite athletes. It will allow the user to add new athletes to the list and to edit existing entries.
  - In your student resources folder, open the starter project called "FavoriteAthlete." The app is already set up with a table view for displaying a list of athletes and a form for collecting information about an individual athlete.
- **Step 1 Make a Plan**
  - Based on the app description, think through the following questions and write down your answers in a brief outline:
    - What model object(s) will you need for this app?
    - What properties will the model object(s) need?
    - In addition to the view controller provided in the starter project, what other controllers will you need?
    - What properties and functions will the controllers need?
  - Did you come up with a plan for moving forward? Your design may differ slightly from the steps in this lab, and that's OK. In app design, there's no one right way to do things. But for the purposes of this project, you'll use the following model and controllers:
    - The Athlete model will store information, through properties, for the athlete's name, age, league, and team.
    - The AthleteTableViewController will handle the view associated with displaying the user's list of favorite athletes.
    - The AthleteFormViewController will handle the views and user input associated with entering and editing athlete information.
- **Step 2 Examine the Starter Project**
  - Take a look at the view controllers in the storyboard. You'll see a table view controller that has two segues to a form. One segue has an identifier of AddAthlete and originates from an Add button. The other segue has an identifier of EditAthlete and originates from a table view cell.
  - The form has a Save button that you'll use to trigger an unwind segue to bring the user back to the table view.
  - The table view controller's subclass has already been created and set as AthleteTableViewController. At the moment, you'll see an error in this file. That's because it references a type, Athlete, that hasn't been created yet.
- **Step 3 Create the Model**
  - Create a new Swift file named "Athlete."
  - In the file, create an Athlete struct that has the following properties: name, age, league, and team.
  - Add a computed property description of type String that uses the four properties to return a phrase describing the athlete, as in the following:

    - ```swift
        var description: String {
            return "\(name) is \(age) years old and plays for the \(team) in the \(league)."
        }
      ```

- **Step 4 Create and Implement a View Controller Subclass**
  - Create a new Cocoa Touch Class file that subclasses UIViewController and is named “AthleteFormViewController.” In the Main storyboard, use the Identity inspector to set the class of the form scene to your new subclass.
  - Add a variable property athlete of type Athlete? to your AthleteFormViewController class. Why is this variable optional? It will be nil when the user comes to the screen to create a new athlete, and it will have a value when the user comes to the screen to edit an athlete.
  - Create a custom initializer with the signature init?(coder: NSCoder, athlete: Athlete?). Assign self.athlete to your instance variable and call the super implementation. Satisfy the compilation error that this creates by using Xcode's fix-it suggestion.

    - ```swift
        class AthleteFormViewController: UIViewController {
            var athlete: Athlete?
            
            init?(coder: NSCoder, athlete: Athlete?) {
                self.athlete = athlete
                super.init(coder: coder)
            }
            
            required init?(coder: NSCoder) {
                // fatalError("init(coder:) has not been implemented")
                super.init(coder: coder)
            }
            ...
      ```

  - Create an updateView() method and call it in the viewDidLoad() method.
  - Add outlets from the storyboard to all the text fields in AthleteFormViewController. In the same file, add an action for the Save button.
  - In updateView(), unwrap the athlete property and check whether it contains an Athlete object. If so, use the object to update the text fields with the athlete's information.

    - ```swift
        func updateView() {
            guard let theAthlete = athlete else { return }
            nameTextField.text = theAthlete.name
            ageTextField.text = "\(theAthlete.age)"
            leagueTextField.text = theAthlete.league
            teamTextField.text = theAthlete.team
        }
      ```

  - Inside the the action for the Save button, create an Athlete object by unwrapping the text from the text fields and passing it to the Athlete initializer, as follows:​

    - ```swift
        guard let name = nameTextField.text,
            let ageString = ageTextField.text,
            let age = Int(ageString),
            let league = leagueTextField.text,
            let team = teamTextField.text else {return}
         
        athlete = Athlete(name: name, age: age, league: league, team: team)
      ```

- **Step 5 Finish Implementing AthleteTableViewController**
  - In the starter project for this lab, most of the AthleteTableViewController class has already been implemented. You'll learn how to set up a table view on your own in a later lesson. For now, you simply need to create the functionality for informing the next view controller which athlete was tapped and for receiving information about an added or edited athlete.
  - Create an @IBSegueAction from the AddAthlete segue by Control-dragging from the segue arrow between the two controllers into AthleteTableViewController. Name it `addAthlete` with no arguments. Do the same for the EditAthlete segue, naming the action `editAthlete` and setting Arguments to `Sender`.
  - In the addAthlete method, instantiate and return a new instance of AthleteFormViewController.
  - When the editAthlete method is called, the sender parameter will be the cell that was touched. Since you haven't yet learned about table views, this next part might be tricky, but see if you can follow along. Your Athlete objects are stored in an array called athletes in the table view controller. The index of each athlete in the array corresponds to the index of the table cell displaying that athlete. UITableView provides a method to look up a cell's IndexPath, which you can use to look up the corresponding Athlete in your athletes array.
  - In the editAthlete method, unwrap the `sender` and cast it to a `UITableViewCell`, then use UITableView's `indexPath(for:)` method to get the IndexPath for the cell. The index path method returns an optional, so you can include that call in your unwrapping logic. Once you have the index path, you can access its row property, which translates to the index of the athlete in your athletes array. Your implementation might look like the following:​

    - ```swift
        let athleteToEdit: Athlete?
        if let cell = sender as? UITableViewCell,
          let indexPath = tableView.indexPath(for: cell) {
            athleteToEdit = athletes[indexPath.row]
        } else {
            athleteToEdit = nil
        }
         
        return AthleteFormViewController(coder: coder, athlete: athleteToEdit)
      ```

  - Next, you'll enable the table view controller to receive an Athlete object back from the AthleteFormViewController in the event that a new athlete has been added or an existing athlete has been edited. You will do this with an unwind segue. In AthleteTableViewController, add an @IBAction method that takes a parameter of type UIStoryboardSegue. Inside the body of this method, you'll first need to cast the segue's source view controller as an AthleteFormViewController and unwrap its athlete property with a guard statement.
  - Which index path, if any, was selected before going to AthleteFormViewController? You can use the table view's indexPathForSelectedRow property in combination with conditional binding to discover the path. If the returned index path is successfully unwrapped, the Athlete object coming back is an edited athlete, not a new athlete. You'll need to use the index path to replace the existing athlete in the athletes array. If the returned index path is unsuccessfully unwrapped, the Athlete object coming back is a new athlete—in which case you can simply append the new athlete to the end of the athletes array. Here's how all this works:​

    - ```swift
        @IBAction func unwindToAthleteList(segue: UIStoryboardSegue) {
            guard let athleteFormViewController = segue.source as? AthleteFormViewController,
                  let athlete = athleteFormViewController.athlete else { return }
            if let selectedIndexPath = tableView.indexPathForSelectedRow {
                athletes[selectedIndexPath.row] = athlete
            } else {
                athletes.append(athlete)
            }
        }
      ```

- **Step 6 Perform the Unwind Segue in Storyboard**
  - Finally, you need to create the unwind segue. In the storyboard, Control-drag from the Athlete Form View Controller to the view controller's Exit, then choose your unwind segue: `unwindToAthleteListWithSegue`. Give this segue a name, `segueForm`, by selecting it in the Document Outline and adding the identifier in the Attributes inspector.
  - Now you need to instruct the segue when to execute in the AthleteFormViewController. Do this by calling `performSegue(withIdentifier:sender:)` at the end of your Save button's @IBAction method, passing in the identifier of the unwind segue and self as the sender: `performSegue(withIdentifier: "segueForm", sender: self)`
  - Now run the app and see if you can add and edit your favorite athletes.
  - Congratulations! You've planned and created a simple app using MVC principles. Be sure to save your work to your project folder.

### Lesson 1.4 Scroll Views

- Whenever your app needs to display content or allow manipulation of content that doesn't fit entirely on the screen, it's time for a scroll view. You can see an excellent example in the Photos app. Open Photos on an iOS device and select a photo. At first tap, the picture will fill as much of the screen as possible while maintaining the same aspect ratio. Now imagine you want to inspect a particular detail of the image. Zoom into your picture by double-tapping. Even though the photo is now bigger than the screen — especially if you're using a smaller device — you'll be able to scroll anywhere you like on the image.
- A UIScrollView needs to know two sets of information to function: the position and size of the scroll view, and the size of the content being displayed. Programmatically, these values are stored in the scroll view's frame property and contentSize property, respectively. Both properties can be managed using Auto Layout and Interface Builder.
- For scrolling to happen, the width or height (or both) of the content size must be greater than the frame's width or height. In the layout below, the frame size is represented by the red box. Notice that the content height is greater than the frame height and the content width is the same as the frame width - so this view will only scroll vertically.

  - <img src="./resources/layout_for_scrollView.png" alt="Scroll View" width="300" />

- By adjusting the sizes, you can create scroll views that scroll horizontally, vertically, or both. In addition to the content size requirement, all views you wish to scroll must be subviews of the scroll view.

#### Scroll Views in Interface Builder

- In this lesson, you'll learn how to implement scroll views with the Auto Layout feature of Interface Builder. Imagine you're designing a UI that will allow users to enter their shipping information, perhaps as part of a checkout flow for an online purchase. It will look much like the layout above.
- Start by creating a new Xcode project using the iOS App template. Name the project "ScrollingForm." When creating the project, make sure the interface option is set to "Storyboard" and save it to your project folder.
- **Define Scroll View Frame**
  - In the Main storyboard, find a scroll view in the Objects library and drag it onto the view controller scene. Adjust the size of the scroll view to match the size of the scene's Safe Area.
  - Add constraints to pin all four edges of the scroll view to the edges of the view controller's Safe Area. These Auto Layout constraints apply to the scroll view's Frame Layout Guide and ensure the scroll view size is the same as the view controller's view — whether on an iPhone SE, an iPad Pro, or any device in between. The constraints also tell the scroll view its size and position.
- **Programmatic Constraints to Content Layout Guide**
  - If needed, you can also apply constraints programmatically to a scroll view's Content Layout Guide. In other words, you can not only constrain a scroll view's size and position but also place constraints to control the behavior of a scroll view's content. For example, if you have an image view inside of a scroll view that you want to stay centered when zooming in and out, you can anchor the center of of the image view to the center of the scroll view's Content Layout Guide, as follows:

    - ```swift
        imageView.centerXAnchor.constraints(equalTo: scrollView.contentLayoutGuide.centerXAnchor)
        imageView.centerYAnchor.constraints(equalTo: scrollView.contentLayoutGuide.centerYAnchor)
      ```

  - This is just one example, and while you won't use constraints on a scroll view's Content Layout Guide in this lesson's scrolling form, they can be useful in many scenarios.
- **Define Content View Using Stack View**
  - Next, you'll tell the scroll view the size of the content. For most common layout tasks, the logic to implement a scroll view with Auto Layout will be much easier if you use a dummy view to contain the scroll view’s content. The dummy view's job is to contain the content and, generally, it is not seen by the user. This is sometimes referred to as a content view.
  - To continue your form, you'll use a stack view to contain all your content.
    - Select a vertical stack view from the Object library and add it as a subview of the scroll view.
    - Adjust the size to be the same as the scroll view
    - Pin the edges of the stack view to the Content Layout Guide by Control-dragging from the Stack View in the outline to the Content Layout Guide on the Document Outline. By holding Command while clicking each constraint, you can add them all without the menu closing.

    - <img src="./resources/pin_edges_of_stackView.png" alt="Pin the edges of stack view to the Content Layout Guide" width="400" />

    - In the Document Outline, click the arrow next to Constraints, below Stack View, and expand the outline view's area so you can see the full set of constraints that were just added. Verify that each one's Constant value is set to 0.  The stack view now defines the scroll view's content area.
  - This form should only scroll vertically — not horizontally.
    - Create an Equal Widths constraint between the Stack View and the Frame Layout Guide by Control-dragging from one to the other in the outline.
    - Locate the new constraint in the outline and verify its Constant is 0 and Multiplier is 1. Now the width of the stack view will always be equal to the scroll view's frame, which means it can't scroll horizontally.

#### Creating the Form

- **Start Creating the Form**
  - Think back to the lesson on Auto Layout and stack views. You learned that when a stack view doesn't have a defined width or height, its size is based on its subviews. As you add content to the stack view, it expands to fit all your content. Thus, the content inside the stack view defines the stack view's size. It's only as big as it needs to be.
  - Because stack views grow to fit their content, they are a perfect candidate for holding the content of a scroll view. As the content grows, the scroll view will scroll to meet the needs of the content size.
  - In your ScrollingForm project, you'll add your scrolling content to the stack view. To simplify the process, you'll create a group of views that will allow you to add as many fields as you need. Each group will consist of three views: a UIView (to serve as a background and container), a label, and a text field. The label will tell the user what type of data to enter into the text field. Both will be subviews of the background view.
  - Drag a view from the Object library into the stack view. (Hint: Typing "UIView" in the library's filter is a quick way to find the view object.) Interface Builder is showing that you have layout issues, indicated by a white arrow inside a red circle in the Document Outline, as well as by the red layout indicators in the scene.
  - What's going on? You haven't defined enough size information for the scroll view's content size. For starters, the UIView you added doesn't have an intrinsic, or natural, size. Therefore, the stack view can't determine its size, which means the scroll view can't determine its content size. In the next section, after you've added the rest of your views, you'll make sure that you have enough constraints for the required size definitions.
  - Add a label to the view you just added. Use the alignment guides to place the label in the upper-left of the view.
  - Next, add a text field below the label - again, taking care to use the alignment guides.
  - Adjust the height of the view so that its bottom margin is aligned with the bottom of the text field.
  - Next, adjust the trailing edges of the label and text field so they're aligned with the margin of the view's trailing edge.
- **Add Constraints**
  - Now that your view is nicely aligned, adding constraints should be a breeze. Adding these constraints will resolve the layout issues discussed above by defining the content size of the scroll view. (If you need a quick review, check out the lesson on Auto Layout in Unit 2 of Develop in Swift Fundamentals.)
    - Select the label and click the Add New Constraints tool button. Add a constraint for each edge, '8'.
    - Select the text field and click the Add New Constraints tool button. Since you added a constraint between the text field and the label in the previous step, you only need to add constraints for the three remaining edges: leading: 8, trailing: 8, and bottom: 20
- **Add More Fields to Your Form**
  - If you ran your project now, would you be able to scroll? Why or why not? You'd be correct if you said "no." The content isn't bigger than the scroll view, so scrolling isn't necessary.
  - In the Document Outline, select the view in the stack view and copy it (Command-C). Paste it (Command-V) several times into the stack view, once for every field you want to include in your form. Since your form will collect users' shipping information, you'll probably need the following fields: first name, last name, address line 1, address line 2, city, state, ZIP code, and phone number. Don't forget to modify the labels with descriptions of the field content. 
  - Build and run your app on different devices. On larger devices, you may find there's enough room to display all the views — so the scroll view won't scroll. On smaller devices, such as an iPhone XS, you won't be able to see all the views, but you should be able to scroll to see the rest. If you don't have a physical device small enough, run the app on a small device in Simulator.

#### Keyboard Issues

- When you ran your app on smaller devices, you might have noticed that the keyboard covers some of the text fields while they are being edited.
- Obviously, it's not a good experience if the user can't see the text they're entering. This lesson doesn't provide details about keyboard management with scroll views, but here's a brief explanation that will help you handle the problem.
- iOS sends a notification every time the keyboard is shown. You can add code to listen for the notification and execute a function when the notification is posted. And, because the notification includes the keyboard size, your function can adjust the scroll view so that its contents aren't overlapped by the keyboard. You can also add a listener for another notification that is posted when the keyboard will be hidden, allowing you to reverse the adjustments.
- If you're interested in learning more about notifications, check out the API Reference documentation for [Notification Center](https://developer.apple.com/documentation/foundation/notificationcenter).
- To adjust the size of the content view, you need to create an outlet from the scroll view to its view controller. Then you can add the following code to the ViewController class to fix the keyboard issues:

  - ```swift
      func registerForKeyboardNotifications() {
          NotificationCenter.default.addObserver(self, selector: #selector(keyboardWasShown(_:)), name: UIResponder.keyboardDidShowNotification,
          object: nil)
          NotificationCenter.default.addObserver(self, selector: #selector(keyboardWillBeHidden(_:)), name: UIResponder.keyboardWillHideNotification,
          object: nil)
      }

      @objc func keyboardWasShown(_ notification: NSNotification) {
          guard let info = notification.userInfo,
              let keyboardFrameValue = info[UIResponder.keyboardFrameBeginUserInfoKey]
              as? NSValue else { return }

          let keyboardFrame = keyboardFrameValue.cgRectValue
          let keyboardSize = keyboardFrame.size

          let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height, right: 0.0)
          scrollView.contentInset = contentInsets
          scrollView.scrollIndicatorInsets = contentInsets
      }

      @objc func keyboardWillBeHidden(_ notification: NSNotification) {
          let contentInsets = UIEdgeInsets.zero
          scrollView.contentInset = contentInsets
          scrollView.scrollIndicatorInsets = contentInsets
      }
    ```

- Now add the following line to viewDidLoad(): `registerForKeyboardNotifications()`
- Build and run your app. You should now be able to see what you're typing in any text field.

#### Content Insets and Scroll Indicator Insets

- To ensure that the keyboard does not overlap your content, you used a content inset to change the size of the content area of the scroll view, making room for the keyboard. You'll use this solution often when you want to add padding to the scroll view content so that controllers, toolbars, and keyboards don’t interfere with the user's experience of the content.
- To add padding, use the contentInset property to specify a buffer area around the content of the scroll view. By adding padding, you're making the scroll view's window into its content smaller without changing the size of the scroll view itself or the size of its content.
- The contentInset property is a UIEdgeInsets struct with fields for top, bottom, left, and right.

  - <img src="./resources/contentInset.png" alt="Content Inset" width="400" />

- But there's a problem. Changing the contentInset value has an unexpected side effect when your scroll view displays scroll indicators. For example, the scroll indicators may be drawn behind the keyboard — an unwanted result. To fix this, you'll need to modify the `scrollIndicatorInsets`. Like the contentInset property, the scrollIndicatorInsets property is defined as a UIEdgeInsets struct. You can remedy the situation by setting the scrollIndicatorInsets values to match the contentInset value.

  - ```swift
      let contentInsets = UIEdgeInsets(top: 0.0, left: 0.0, bottom: keyboardSize.height, right: 0.0)
      scrollView.contentInset = contentInsets
      scrollView.scrollIndicatorInsets = contentInsets
    ```

#### The Scroll View Family

- UIScrollView is the parent class of several other classes in UIKit, including UITableView and UICollectionView. These two prominent classes both inherit all the functionality of UIScrollView classes. As you advance to these new topics, you'll want to keep this relationship in mind, since you'll be able to use all the tools you learned in this lesson when you use table views or collection views.

#### Challenge

- Create a horizontal scroll view that displays three of your favorite images.

#### Lab - I Spy

##### Objective

- The objective of this lab is to implement a scroll view on an image view that will enable users to zoom in and pan an image. Your app will have a single view with a single picture.
- Create a new project called "ISpy" using the iOS App template.

##### Step 1 Set Up the Storyboard

- Add a scroll view to the view controller and resize it to fit in the Safe Area. Add constraints to pin the scroll view edges to the Safe Area of the view controller.
- Add an image view as a subview of the scroll view and resize it to fit the scroll view's bounds. Add constraints to pin the edges of the image view to the Content Layout Guide of the scroll view.

  - <img src="./resources/constraints_imageView.png" alt="Constraints of Image View" width="500" />

- Add an image of your choosing to the Assets folder. Set it as the image of the image view.
- Create outlets from the scroll view and image view to the ViewController file.

##### Step 2 Implement the UIScrollViewDelegate

- To use a scroll view for zooming, your view controller needs to conform to UIScrollViewDelegate. You can read more about UIScrollViewDelegate in the [documentation](https://developer.apple.com/documentation/uikit/uiscrollviewdelegate). But for now, go ahead and add the following line of code to the class declaration: `class ViewController: UIViewController, UIScrollViewDelegate {...}`
- In the viewDidLoad() method, set the scroll view's delegate to be the ViewController instance.
- Implement the `viewForZooming()` delegate method, and return the image view in the body of the method.
- Create a new method `updateZoomFor(size: CGSize)` that will update the zoom of the image. Override `viewDidAppear(_:)` and in the body call `updateZoomFor(size:)`, passing in `view.bounds.size`.
- Inside the `updateZoomFor(size:)` method, use the imageView and the size parameter to calculate the scale. Then set the `minimumZoomScale` property of the scrollView to be that scale, as in the following:

  - ```swift
      let widthScale = size.width / imageView.bounds.width 
      let heightScale = size.height / imageView.bounds.height 
      let scale = min(widthScale, heightScale)
      scrollView.minimumZoomScale = scale
      scrollView.zoomScale = scale
    ```

- The code above starts by calculating the scale necessary to show the entire width and height of the image. It then sets the minimum scale to be the smaller of the two (width or height), so that you won't be able to zoom out beyond that. Finally, it sets the initial zoom scale so that the image fits inside the screen.
- Run the app. You should be able to zoom and scroll in all directions. (Use **Option-drag** to zoom in Simulator.)
- Great work! You've made an app that can scroll and zoom in on an image. Can you think of something you want to build where this functionality might be useful? Be sure to save your app to your project folder for future reference.

  - ```swift
      import UIKit

      class ViewController: UIViewController, UIScrollViewDelegate {
          @IBOutlet var scrollView: UIScrollView!
          @IBOutlet var imageView: UIImageView!

          override func viewDidLoad() {
              super.viewDidLoad()
              scrollView.delegate = self
          }

          func viewForZooming(in scrollView: UIScrollView) -> UIView? {
              return imageView
          }
          
          func updateZoomFor(size: CGSize) {
              let widthScale = size.width / imageView.bounds.width
              let heightScale = size.height / imageView.bounds.height
              let scale = min(widthScale, heightScale)
              scrollView.minimumZoomScale = scale
              scrollView.zoomScale = scale
          }
          
          override func viewDidAppear(_ animated: Bool) {
              super.viewDidAppear(animated)
              updateZoomFor(size: view.bounds.size)
          }
      }
    ```

- Tip: When to use super when overriding iOS method
  - It depends on recommendations from the documentation for the superclass's implementation of that method.
  - `viewDidAppear(_:)` - If you override this method, you must call super at some point in your implementation.

### Lesson 1.5 Table Views

- Enter UITableView, one of the most widely used views in iOS. Table views are perfect for efficiently displaying large or small amounts of information and for managing dynamic or static lists of data.
- In this lesson, you'll learn why table views are so popular among iOS developers. This lesson will focus on dynamic table views; the next table view lesson will cover static table views.
- If you've ever used an iOS device, odds are you've encountered a table view, or UITableView. Because of their utility, table views are one of the most ubiquitous views in all of iOS. In the Music app, table views allow you to navigate through your music hierarchy. In Contacts, table views present your contacts in an indexed list. In Settings, table views organize controls into visually distinct groupings. 
- As you work through this lesson, you'll learn how to work with table views by building an emoji dictionary app.
- Before we jump into table views, you'll need to complete a few setup steps — all things you've learned about in previous lessons. To get started, open Xcode and create a new iOS app project called "EmojiDictionary." When creating the project, make sure the interface option is set to "Storyboard." Delete the ViewController file and the view controller scene in the Main storyboard. Now you have a truly blank app canvas to work with.
- The next step is to create your model. Remember that model objects should represent the real-world object you want to display to the user. In this project, you're going to display information about different emoji, so you'll need to create an Emoji model object. Your emoji structure will have the following properties:
  - symbol, a String that holds the emoji symbol
  - name, a String representing the emoji name
  - description, a String describing the emoji
  - usage, a String describing how the emoji is used or its meaning
- For example, an emoji instance might look like the following:

  - ```swift
      {
          symbol: "🐘",
          name: "Elephant",
          description: "A gray elephant.",
          usage: "good memory"
      }
    ```

- Before looking at the code segment below, create a new Swift file called Emoji.swift and try to create a struct to represent an emoji on your own.

  - ```swift
      struct Emoji {   
          var symbol: String
          var name: String
          var description: String
          var usage: String
      }
    ```

- Now that your model is set up, you can dive into table views.

#### Anatomy of a Table View

- A table view is an instance of the UITableView class. It may be responsible for displaying one data object or tens of thousands of data objects. To accommodate many rows of data, table views present data in a scrolling, single-column list, with the option to divide the rows into sections or groups. Each section can have a header above the first item and a footer below the last item. The table view itself can also have its own header and footer.

##### Table View Controllers

- There are two ways to add a table view to your projects, depending on how you will use the table view.
- One approach is to add a table view instance directly to a view controller's view. In this scenario, the view controller may manage other views in addition to the table view. This means you're responsible for managing the position and size of the table view. You're also responsible for setting the data source and delegate object(s). (You'll learn about the data source object and the delegate object later in this lesson.)
- The second approach is to use a table view controller. UITableViewController is a view controller subclass that manages a single table view instance. In this approach, the table view takes up the entire view, and you can't adjust the table view's size. The table view controller also acts as the data source and delegate of the table view.
- While you may find uses for both approaches, table view controllers provide some additional functionality. If you were to use the first approach and also wanted one of these features, you'd have to write additional code. For example, when using a table view controller, the table view's scroll view will always adjust to accommodate the keyboard when necessary. If you were to use a table view added to a regular view controller, you would have to write code to monitor when the keyboard is presented and adjust the table view's content insets accordingly. 
- Except in special cases, you'll likely find that it's best to use a table view controller.
- Back in your EmojiDictionary project, open the Main storyboard and drag a navigation controller from the Object library onto the scene. You'll notice that a second view controller, a table view controller, automatically came along for the ride.
- Next, set the navigation controller to be the initial view controller by selecting the navigation controller and checking "Is Initial View Controller" in the Attributes inspector. Then, update the title in the navigation bar from "Root View Controller" to "Emoji Dictionary."
- Create a new Cocoa Touch Class called `EmojiTableViewController` as a subclass of `UITableViewController`. Back in your storyboard, select the table view controller and assign `EmojiTableViewController` as its custom class in the Identity inspector(1).
- Build and run you app. At this point, you should see an empty table view inside a navigation controller.

  - <img src="./resources/tableViewController.png" alt="Table View Controller" width="500" />

##### Table View Style

- Table views come in three styles: plain, grouped, and inset grouped.
  - The plain style is the default. In a plain table view, rows can be separated into labeled sections with an optional index along the right edge of the table (such as the alphabet index seen in Contacts). Each section immediately follows the previous section with no spacing, creating an unbroken list.
  - In a grouped table view, rows are displayed in visually distinct groups, or sections, with spacing in between - and without an index option.
  - An inset grouped table view is similar to grouped, but each visual group in inset on both sides of the view with rounded corners, providing a very pronounced separation between sections.

##### Table View Cells

- Every row in a table view is represented with a table view cell, a UITableViewCell instance. Cells are reusable views that can display text, images, or any other UIView. In addition to the cell content, each cell has an optional accessory view (more on accessory views later).

  - <img src="./resources/table_view_cell_non_editing.png" alt="Table View Cell - Non-Editing Mode" width="400" />

- In addition to the default, non-editing mode, table views can enter an editing mode, which enables users to insert, delete, or reorder cells. In editing mode, the cell content size shrinks and any accessory view disappears to allow room for the editing and reorder controls.

  - <img src="./resources/table_view_cell_editing.png" alt="Table View Cell - Editing Mode" width="400" />

- The UITableViewCell class defines three properties for cell content. 
  - `textLabel` and `detailTextLabel` are UILabels for the title and subtitle (or additional detail). 
  - `imageView` is a UIImageView for an image.
- These three properties have existed since the release of the iOS 2.0 SDK in 2009. UIKit has evolved since then, and table view cell configurations were introduced in iOS 14.0. Cell configurations let you describe what a cell should contain, rather than directly updating the cell's views yourself. The cell is responsible for taking the information in a configuration and updating its own views. This extra abstraction gives the cell more control over how to display its contents and lets you focus on the data you want it to display. (Also, the support for the textLabel, detailTextLabel, and imageView properties will be deprecated in a future version of iOS, so you should use cell configurations when possible.)
- To configure a cell, request its defaultContentConfiguration and set the properties of the returned content configuration. Then update the cell's contentConfiguration property with the updated configuration. The configuration has properties for text, secondaryText, image, and more to adjust the presentation of elements in a cell. You'll see cell content configurations in action shortly.
- The UIKit framework defines four standard cell styles that you can select in Interface Builder or programmatically, using the enum UITableViewCell.CellStyle. Each style supports its own combination of properties and has its own layout.
- | Storyboard Name | Programmatic Enum Name | Displays | Description |
  |---|---|---|---|
  | Basic | .default | textLabel, imageView | A label on the left side of the cell with black, left-aligned text; options leading image view. <img src="./resources/table_basic.png" alt="Basic" width="200" />|
  | Subtitle | .subtitle | textLabel, detailTextLabel, imageView | A label across the top with black, left-aligned text; a second label below in smaller, gray, left-aligned text; optional leading image view.  <img src="./resources/table_subtitle.png" alt="Subtitle" width="200" />|
  | Right Detail | .value1 | textLabel, detailTextLabel, imageView | A label on the left side of the cell with black, left-aligned text; a label on the right side with black(storyboard) or gray (programmatic), right-align text; optional leading image view. <img src="./resources/table_right_detail.png" alt="Right Detail" width="200" />|
  | Left Detail | .value2 | textLabel, detailTextLabel | A label on the left side of the cell with small, black, right aligned text, followed by a detail label with small, black (storyboard) or blue (programmatic), left-align text. <img src="./resources/table_left_detail.png" alt="Left Detail" width="200" />|
-  When the table is in default (view-only) mode, cells can display an accessory view, which you can also select in Interface Builder or programmatically, using the enum `UITableViewCell.AccessoryType`. The iOS SDK defines five standard types of accessory views, some of which can respond to user touches, like a button. With a delegate method, you can respond and execute code when the user taps the accessory view. You'll learn about the delegate later in this lesson.
No matter which accessory view is displayed, your code is responsible for responding to the user's tap on a cell or its accessory view. Accessory views are just indicators to the user about the nature of the cell.
  - | Storyboard Name | Programmatic Enum Name | Description | Image |
    |---|---|---|---|
    | None | .none |  No accessory view is displayed; content view takes up the entire cell. | <img src="./resources/accessoryType_none.png" alt="" width="50" /> |
    | Disclosure Indicator | .disclosureIndicator | A chevron. Tapping the cell typically triggers a show segue. The accessory view does not track touches. | <img src="./resources/accessoryType_disclosure.png" alt="" width="50" /> |
    | Detail | .detailButton | An info button. The accessory view tracks touches; tapping it typically displays additional information about the cell's contents. | <img src="./resources/accessoryType_detail.png" alt="" width="50" /> |
    | Detail Disclosure | .detailDisclosureButton | An info button and a chevron. Tapping the cell typically triggers a show segue. The accessory view tracks touches and typically displays additional information about the cell's contents. | <img src="./resources/accessoryType_detail_disclosure.png" alt="" width="50" /> |
    | Checkmark | .checkmark | A checkmark. The accessory view does not track touches. | <img src="./resources/accessoryType_checkmark.png" alt="" width="50" /> |
- Return to your EmojiDictionary project. In the table view controller scene, notice there is one cell under the Prototype Cells header. If you select the cell and open the Attributes inspector, you'll see many adjustable options, including Style. Feel free to select the different styles to see the options. When you're done, select the Subtitle style. You will use the labels to display the emoji symbol and name on the title line and the description as the subtitle. This style gives you the most room for both of your labels.

##### Table View Readability Margins

- By default, table view cells take up the entire width of the table view. This may cause readability issues on large screens: It's hard to read a very long line of text, and short lines can be awkward when the label is far from its accessory view. But there's a fix. With the table view selected in your storyboard, navigate to the `Size inspector` and find and check the `Follow Readable Width` option. Now, the width of your table view cells will stay within readable margins.
- You can also set this option programmatically by setting the `cellLayoutMarginsFollowReadableWidth` property to true. You can do this in the viewDidLoad() method of EmojiTableViewController.

#### Index Paths

- Many of the API methods for table views either accept an index path as a parameter or return an index path. As you might guess, an index path points to a specific row in a specific section of a table view. You can access these values through the index path's row and section properties. Just like array indices, these values are zero-indexed.

  - <img src="./resources/table_index_path.png" alt="Index Paths" width="200" />

#### Arrays and Table Views

- Table views are ideal for displaying a collection of similar data and are typically backed by a collection of model objects. This collection is usually an array — though it's possible to use another collection, such as a dictionary. Why is an array the best match with a table view? First, ordering of an array is guaranteed and stable. Also, arrays have a property (count) to query how many objects they contain, which can then inform the table view how many rows to display. In addition, arrays and index path values are both zero-based, which simplifies accurate addressing.
- To display a list of emoji in EmojiDictionary, you'll need to create an array of Emoji and add it as a property to your EmojiTableViewController. Feel free to create your own list or use the array provided below. (While it could be argued that the plural of “emoji” is “emoji,” for the sake of clarity — always a programmer's goal—“emojis” is the best name for your array.)

  - ```swift
      var emojis: [Emoji] = [
          Emoji(symbol: "😀", name: "Grinning Face", description: "A typical smiley face.", usage: "happiness"),
          Emoji(symbol: "😕", name: "Confused Face", description: "A confused, puzzled face.", usage: "unsure what to think; displeasure"),
          Emoji(symbol: "😍", name: "Heart Eyes", description: "A smiley face with hearts for eyes.", usage: "love of something; attractive"),
          Emoji(symbol: "🧑‍💻", name: "Developer", description: "A person working on a MacBook (probably using Xcode to write iOS apps in Swift).", usage: "apps, software, programming"),
          Emoji(symbol: "🐢", name: "Turtle", description: "A cute turtle.", usage: "something slow"),
          Emoji(symbol: "🐘", name: "Elephant", description: "A gray elephant.", usage: "good memory"),
          Emoji(symbol: "🍝", name: "Spaghetti", description: "A plate of spaghetti.", usage: "spaghetti"),
          Emoji(symbol: "🎲", name: "Die", description: "A single die.", usage: "taking a risk, chance; game"),
          Emoji(symbol: "⛺️", name: "Tent", description: "A small tent.", usage: "camping"),
          Emoji(symbol: "📚", name: "Stack of Books", description: "Three colored books stacked on each other.", usage: "homework, studying"),
          Emoji(symbol: "💔", name: "Broken Heart", description: "A red, broken heart.", usage: "extreme sadness"), 
          Emoji(symbol: "💤", name: "Snore", description: "Three blue \'z\'s.", usage: "tired, sleepiness"),
          Emoji(symbol: "🏁", name: "Checkered Flag", description: "A black-and-white checkered flag.", usage: "completion")
      ]
    ```

##### Cell Dequeueing

- Before diving into displaying this data, you first need to learn about cell dequeueing. It's possible to create a table view that's responsible for displaying hundreds of thousands of model objects. Each of these objects will be displayed by a cell, and each could have multiple views. Every view displayed onscreen must exist in a device's finite memory. If the table view were to try to initialize a cell for each object, a large list would quickly cause the device to run out of memory, resulting in a crash.
- To address this issue, table views only load the visible cells — plus a few more to make sure scrolling stays smooth. As the user scrolls through a table view, cells leave the visual field, and others are displayed at the opposite end of the device. Typically, the cells entering the visual field share the same layout as the ones that left the visual field moments ago. Could you take the cell that just left and reuse it as the cell that's about to enter?
- In fact, this process — called dequeueing — is precisely how table views manage their cells. You can register each cell type in your table view with a `reuseIdentifier` string. The table view can then manage a stockpile of cells for each reuse identifier. When you want a cell, you use the table view instance method `dequeueReusableCell(withIdentifier:for:)`, passing in the reuse identifier for the desired cell type. This method asks the table view instance to provide a matching cell that's available for reuse: `let cell: UITableViewCell = tableView.dequeueResuableCell(withIdentifier: "Cell", for: indexPath)`
- You can register your cells in Interface Builder by selecting the prototype cell and adding a value to the reuse identifier field in the Attributes inspector. In EmojiDictionary, select your table view cell. Then, navigate to the Attributes inspector and add `EmojiCell` as the Identifier. Your cell is now registered.
- Through the process of dequeueing, the table view only creates and loads as many cells as absolutely necessary, rather than one cell for every piece of information displayed. Dequeueing saves memory and allows for a smooth flow when scrolling through table views — providing the best possible user experience.

#### Table View Protocols

- A dynamic table view object must have a data source object and may or may not have a delegate object. Following the MVC design model you learned about in an earlier lesson, the data source acts as an intermediary between the table view and your app's data model. The optional delegate manages the appearance (minus the actual cells) and the behavior of the table view.
- Since the UITableView class itself doesn't have the methods, properties, or data required to configure its own contents, it delegates that responsibility to other objects. The data source and delegate are often, but not always, the same object. Typically this object is your custom subclass of UITableViewController. The responsibilities of the data source and delegate are defined in the UITableViewDataSource and UITableViewDelegate protocols.
- As a reminder, protocols define methods and properties for the objects conforming to the protocol to implement. Now that you understand the general difference between the two table view protocols, you'll explore the specific methods defined by them.

#### Table View Data Source

- The data source, which adopts the `UITableViewDataSource` protocol, is responsible for providing the necessary data to the table view object.
- Take a moment to think about the minimum information a table view would need to display its contents. It needs to know how many things to display (i.e. how many rows are in the table) and what goes in each row (i.e. the contents of each cell).
- To provide this information, you implement the methods outlined in UITableViewDataSource.
- For now, you'll focus on three data source methods that provide the number of sections in the table view, the number of rows in each section, and the cell to display for each index path.

  - ```swift
      optional func numberOfSections(in tableView: UITableView) -> Int
      func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int
      func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell
    ```

- When the table view loads or reloads its data, it queries the data source by calling each of these methods (sometimes multiple times) to request information for the visible rows. As the user scrolls through the table view and different rows become visible, the table view continues to query the data source for information to fill the new rows about to be displayed. The data source is responsible for returning the requested information based on the values of the parameters passed into the methods.
- In the next three sections, you'll explore the parameters of each function and your job, as the programmer, when implementing these functions.
- **Number of Sections**
  - In `numberOfSections(in:)`, you return as an Int the number of sections you want the table view to display. You probably noticed in the code above that this function has the optional modifier. If you choose not to write this function, the table view will display one section.
- **Number of Rows in a Section**
  - Once the table view knows how many sections to display, it needs to know how many rows there will be in each section. The `tableView(_:numberOfRowsInSection:)` method has two parameters: the table view requesting the information and the section in question. Based on these parameters, your job is to return an Int representing the number of rows for the given section. This required method will be called for every section in your table view.
- **Cell for Row at Index Path**
  - The table view now knows how many sections to display and how many rows there should be in each section. `tableView(_:cellForRowAt:)` provides you with an index path, and in the body of the method you dequeue, configure, and return a cell for the table view to display.
  - While different situations will call for different implementations of this method, most will adhere to the following steps:
    1. Fetch the correct cell type by dequeueing a cell.
    2. Fetch the model object to be displayed.
    3. Configure the cell's properties with the model object's properties — in other words, set views (labels, image views, etc.) based on the model object.
    4. Return the fully configured cell.
