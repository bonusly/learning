# Factory Design Pattern (adapted for Ruby)
Defines an interface for creating an object, but lets subclasses decide which class
to instantiate. Factory Methods let a class defer instantiation to subclasses.

## Problem:
We want a united interface for creating similar types of objects without having to know anything about the objects themselves.

## Solution:
This pattern is less common in Ruby because of how flexible modules are, but they can still be useful for abstracting that complex logic.

## Example:
Pretend we went a way to create alerts without having to know anything about the types of alerts that exist.

We just want to do something like:
```ruby
@alerts = AlertFactory.all_alerts_for(user)

@alerts.each(&:show)

class AlertFactory
  attr_reader :user

  def self.all_alerts_for(user)
    new(user).all_alerts
  end

  def initialize(user)
    @user = user
  end
  
  private
 
  def all_alerts
    alerts = []
    
    if user.company.integration_error?
      alerts << build(message: user.company.integration_error_message, severity: :warning, for_role: :admin)
    end
    
    if user.is_being_judged?
      if user.is_great?
        alerts << build(message: 'You\'re great!', severity: :success)
      else
        alerts << build(message: 'You\'re terrible.', severity: :error, features: %i[persistent])
      end
    end
    
    alerts
  end
  
  def build(message:, severity: :note, for_role: :all, features: [])
    return if user.company_admin? || for_role != :admin
    
    alert_class_for(severity).new(message).tap do |message|
      message.extend(Alert::Persistence) if features.include? :persistence
    end
  end
  
  def alert_clas_for
    case severity
    when :note
      NoteAlert
    when :error
      ErrorAlert
    default
      SuccessAlert
    end
  end
end
```

## Concequences:
- Allows for complex logic to be abstracted away from both the classes they create and the client.
- Can result in overengineered logic when the base logic is simple.  Usually an indication that the client already knows exactly what it wants and doesn't need to delegate that responsibility.
- Can make code more complicated if the factory returns unrelated objects.  Factories usually use abstract classes or modules to enforce similar behavior across objects.
- Can be used to enfoce or wrap a group of objects in a specific functionality.  For example, a factory can return a three dimensional shape depending on the two dimensional shape passed into it. 
