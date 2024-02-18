# Leveraging Service Objects and Concerns in Refactoring Rails Model Callbacks

![Dog sitting in front of the laptop](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/odyf6uug65jkfxmfo54f.gif)

Active Record callbacks in Rails are a double-edged sword, offering automation while potentially leading to tangled code. Knowing when to use them and when to avoid them is crucial bacuse they are powerful. Read more about ActiveRecord Callbacks from the [official docs](https://edgeguides.rubyonrails.org/active_record_callbacks.html)

## Leveraging Active Record Callbacks
* Streamlined Workflows: Utilize callbacks for sequential tasks in an object's life cycle, such as encrypting passwords or updating caches, ensuring a smoother operational flow.
* Enhanced Code Organization: Utilize callbacks, potentially in tandem with ActiveRecord Concerns, to effectively consolidate repetitive tasks across diverse models, fostering code readability.
* Centralized External Interactions: Employ callbacks to centralize interactions with external services, preventing code scattering and promoting a more structured approach.

## Cautionary Measures
* Combatting Complexity: Over-reliance on callbacks can obfuscate code flow, leading to challenges in debugging and comprehension.
* Unforeseen Impacts: Callbacks might inadvertently trigger unintended side effects, especially when altering associated models.
* Testing and Maintenance Hurdles: Extensive callback usage can complicate testing procedures and make maintenance a daunting task.

---

## Callback Usage Examples:

Example - Fictional code example with multiple callbacks
{% highlight ruby %}
```
class Order < ApplicationRecord
  belongs_to :user

  after_create :send_order_confirmation_email
  after_save :update_user_activity_log
  after_commit :generate_invoice_pdf, on: [:create]

  list goes on...
  
  private
  
  def send_order_confirmation_email
    # Send confirmation email logic...
    # Send confirmation email logic...
    # Send confirmation email logic...
    # Send confirmation email logic...
    # Send confirmation email logic...
    # Send confirmation email logic...
  end
  
  def update_user_activity_log
    # Update user's activity log logic...
    # Update user's activity log logic...
    # Update user's activity log logic...
    # Update user's activity log logic...
    # Update user's activity log logic...
    # Update user's activity log logic...
  end
  
  def generate_invoice_pdf
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
    # Generate invoice PDF logic...
  end
end
```
{% endhighlight %}

The `Order` model currently hosts callback methods, entangled with method logic defined within each function. Additionally, certain callbacks are detached from the core functionality of the `Order` object. This setup complicates testing and debugging, potentially making it challenging to identify and rectify issues.

To address these challenges and enhance maintainability, we can employ service objects and concerns. By extracting the logic from callbacks, we can create **reusable** code blocks applicable across multiple models. This restructuring ensures streamlined testing and debugging processes, fostering a more efficient and adaptable codebase.

Now, let's refactor this example. Here's how we'd do it: 

## Creating Service Objects

```ruby
  class Mailer
    def self.send_confirmation_email(order)
      # Send confirmation email logic here...
    end
  end
  
  class Pdf
    def self.generate_invoice(order)
      # Generation of pdf logic goes here...
    end
  end
```

## Creating Concerns and Using Service Objects

```ruby
  module Logable
    extend ActiveSupport::Concern
  
    included do
      after_commit :update_user_activity_log
    end
  
    private
  
    def update_user_activity_log
      # Update current_user log here...
    end
  end
```

## Refactored version of the model

```ruby
  class Order < ApplicationRecord
    belongs_to :user
  
    include Logable
  
    after_create :generate_invoice, :send_confirmation_email
  
    private
  
    def generate_invoice
      Pdf.generate_invoice(self)
    end
  
    def send_confirmation_email
      Mailer.send_confirmation_email(self)
    end
  end
```

In the process of refactoring the `Order` model, we've optimized the code by introducing service objects dedicated to distinct tasks, such as sending emails and generating PDFs. This strategic approach not only improves the model's testability, readability, and maintainability but also aligns with recommended Rails development practices.

The incorporation of concern `Logable` together with the `Mailer` and `Pdf` services have played key roles in organizing shared functionalities within the model. This deliberate structuring results in a more transparent and succinct model, fostering better modularity, streamlined testing processes, and an overall more sustainable codebase.

Please note that the examples provided are hypothetical and created for illustrative purposes, not based on actual implementations in a specific system or application. They serve as demonstrations to showcase best practices or potential solutions within a coding context.

Happy codding!
