# Loqate

[![Gem Version](https://badge.fury.io/rb/loqate.svg)](https://badge.fury.io/rb/loqate)
[![Build Status](https://travis-ci.org/wilsonsilva/loqate.svg?branch=master)](https://travis-ci.org/wilsonsilva/loqate)
[![Maintainability](https://api.codeclimate.com/v1/badges/5c1414d5dedc68c15533/maintainability)](https://codeclimate.com/github/wilsonsilva/loqate/maintainability)
[![Test Coverage](https://api.codeclimate.com/v1/badges/5c1414d5dedc68c15533/test_coverage)](https://codeclimate.com/github/wilsonsilva/loqate/test_coverage)
[![Security](https://hakiri.io/github/wilsonsilva/loqate/master.svg)](https://hakiri.io/github/wilsonsilva/loqate/master)
[![Inline docs](http://inch-ci.org/github/wilsonsilva/loqate.svg?branch=master)](http://inch-ci.org/github/wilsonsilva/loqate)

Client to address verification, postcode lookup, & data quality services from Loqate.

## Table of contents
- [Installation](#installation)
- [Usage](#usage)
  - [Getting started](#getting-started)
  - [Bang methods](#bang-methods)
    - [Example of using non-bang method](#example-of-using-non-bang-method)
    - [Example of using bang method](#example-of-using-bang-method)
  - [Address API](#address-api)
    - [Finding addresses](#finding-addresses)
    - [Retrieving the details of an address](#retrieving-the-details-of-an-address)
  - [Geocoding API](#geocoding-api)
    - [Finding directions](#finding-directions)
    - [Geocoding a location](#geocoding-a-location)
    - [Finding a country based on coordinates](#finding-a-country-based-on-coordinates)
    - [Finding nearest places](#finding-nearest-places)
  - [Phone API](#phone-api)
    - [Validating a phone number](#validating-a-phone-number)
  - [Email API](#phone-api)
    - [Validating an email address](#validating-an-email-address)
    - [Validating multiple email addresses](#validating-multiple-email-addresses)
  - [Bank API](#bank-api)
    - [Retrieving the details of a bank branch](#retrieving-the-details-of-a-bank-branch)
    - [Validating a bank account](#validating-a-bank-account)
    - [Validating an international bank account](#validating-an-international-bank-account)
    - [Validating multiple bank accounts](#validating-multiple-bank-accounts)
    - [Validating a card](#validating-a-card)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'loqate'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install loqate

## Usage

### Getting started

Loqate provides multiple APIs. And each API provides several services. This gem exposes these APIs through
an API gateway.

To get started, initialize an API gateway with [your API key](https://account.loqate.com/account#/):

```ruby
gateway = Loqate::Gateway.new(api_key: '<YOUR_API_KEY>')
```

### Bang methods

Most methods have a bang and a non-bang version (e.g. `gateway.address.find` and `gateway.address.find!`).
The non-bang version will either return a `Loqate::Success` or an `Loqate::Failure`. The bang version will
either return the desired resource, or it will raise an exception.

#### Example of using non-bang method

```ruby
result = gateway.address.find(text: 'EC1Y 8AF', country: 'GB', limit: 5)

if result.success?
  addresses = result.value
  addresses.first.id # => 'GB|RM|B|8144611'
else
  error = result.error
  puts "Error retrieving the address: #{error.description}. Resolution: #{error.resolution}"
end

result.failure? # => false
```

#### Example of using bang method

```ruby
begin
  addresses = gateway.address.find!(text: 'EC1Y 8AF', country: 'GB', limit: 5)
  addresses.first.id # => 'GB|RM|B|8144611'
rescue Loqate::Error => e
  puts "Error retrieving the address: #{e.description}. Resolution: #{e.resolution}"
end
```

It is recommended that you use the bang methods when you assume that the data is valid and do not expect validations
to fail. Otherwise, use the non-bang methods.

### Address API

The Address API consists of two main API requests:
[Find](https://www.loqate.com/resources/support/apis/Capture/Interactive/Find/1/) request is used to narrow down a
possible list of addresses;
and a [Retrieve](https://www.loqate.com/resources/support/apis/Capture/Interactive/Retrieve/1/) request is used to
retrieve a fully formatted address.

A typical address search is made up of a series of `find` requests, followed by a `retrieve` based on the user
selection.

#### Finding addresses

```ruby
addresses = gateway.address.find!(text: 'EC1Y 8AF', country: 'GB', limit: 5)
addresses.first.id # => 'GB|RM|B|8144611'
```

#### Retrieving the details of an address

```ruby
address = gateway.address.retrieve!(id: 'GB|RM|B|8144611')

address.city        # 'London' 
address.line1       # '148 Warner Road'
address.postal_code # 'E17 7EA'
```

### Geocoding API

The Geocoding API exposes endpoints to perform geocoding operations.

#### Finding directions
```ruby
directions = gateway.geocoding.directions!(
  start: [51.5079532, -0.1266053],
  finish: [51.5078677, -0.1266825]
)
directions.first.description # => 'Leave from Strand/A4'
```

#### Geocoding a location
```ruby
location = gateway.geocoding.geocode!(country: 'US', location: '90210')
location.name # => 'Beverly Hills, CA 90210'
```

#### Finding a country based on coordinates
```ruby
country = gateway.geocoding.position_to_country!(latitude: 52.1321, longitude: -2.1001)
country.country_name # => 'United Kingdom'
```

#### Finding nearest places
```ruby
nearest_places = gateway.geocoding.retrieve_nearest_places!(
  centre_point: [-33.440113067627, 149.578567504883],
  maximum_items: 5,
  maximum_radius: 10,
  filter_options: 'HideVillages'
)
nearest_places.first.location # => 'South Bathurst, NSW, Australia'
```

### Phone API

The Phone API consists of a single API request:
[Validate](https://www.loqate.com/resources/support/apis/PhoneNumberValidation/Interactive/Validate/2.2/) which starts
a new phone number validation request.

#### Validating a phone number

```ruby
phone_validation = gateway.phone.validate!(phone: '+447440029210', country: 'GB')
phone_validation.phone_number # => '+447440029210'
phone_validation.valid?       # => true
```

### Email API

The Email API consists of two main API requests:
[Validate](https://www.loqate.com/resources/support/apis/EmailValidation/Interactive/Validate/2/) request to validate
a single email adress; and a
[Batch Validate](https://www.loqate.com/resources/support/apis/EmailValidation/Batch/Validate/1.2/) request to
validate multiple emails at once.

#### Validating an email address
```ruby
email_validation = gateway.email.validate!(email: 'person@gmail.com')
email_validation.valid? # => true
```

#### Validating multiple email addresses

```ruby                                                                    
email_validations = gateway.email.batch_validate!(emails: %w[person@gmail.com])
email_validation  = email_validations.first
email_validation.valid? # => true
```

### Bank API

The Bank API exposes endpoints to validate bank accounts, cards and finding bank branches by sort code.

#### Retrieving the details of a bank branch
```ruby
bank_branch = gateway.bank.retrieve_by_sortcode!(sort_code: '404131')
bank_branch.bank # => HSBC UK BANK PLC
```

#### Validating a bank account
```ruby
account_validation = gateway.bank.validate_account(account_number: '51065718', sort_code: '404131')
account_validation.correct? # => true
```

#### Validating an international bank account

```ruby
account_validation = gateway.bank.validate_international_account!(iban: 'GB03 BARC 201147 8397 7692')
account_validation.correct? # => true
```

#### Validating multiple bank accounts
```ruby
accounts_validations = gateway.bank.batch_validate_accounts!(
  account_numbers: %w[51065718 12001020],
  sort_codes: %w[40-41-31 083210]
)
accounts_validations.first.correct?  # => true
accounts_validations.second.correct? # => false
```

#### Validating a card
```ruby
card_validation = gateway.bank.validate_card!(card_number: '4024007171239865')
card_validation.card_type # => VISA
```
   
## Development

After checking out the repo, run `bin/setup` to install dependencies, configure git hooks and create support files.

You can also run `bin/console` for an interactive prompt that will allow you to experiment.

Then add your Loqate API key to `.api_key`. It will be used by RSpec and VCR, but not stored in the codebase. 

The health and maintainability of the codebase is ensured through a set of
Rake tasks to test, lint and audit the gem for security vulnerabilities and documentation:

```
rake bundle:audit          # Checks for vulnerable versions of gems 
rake qa                    # Test, lint and perform security and documentation audits
rake rubocop               # Lint the codebase with RuboCop
rake rubocop:auto_correct  # Auto-correct RuboCop offenses
rake spec                  # Run RSpec code examples
rake verify_measurements   # Verify that yardstick coverage is at least 100%
rake yard                  # Generate YARD Documentation
rake yard:junk             # Check the junk in your YARD Documentation
rake yardstick_measure     # Measure docs in lib/**/*.rb with yardstick
```

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/wilsonsilva/loqate.

## License

See [LICENSE](https://github.com/wilsonsilva/loqate/blob/master/LICENSE).
