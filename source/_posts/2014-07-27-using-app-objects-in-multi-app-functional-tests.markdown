---
layout: post
title: "Using app objects in multi app functional tests"
date: 2014-07-27 18:40:04 +0530
comments: true
categories: functional-tests capybara app-objects
---

It is better to create small modular apps instead of single monolithic application. In [Bahmni EMR](http://www.bahmni.org) we have small apps for Patient Registration, Consultation, etc. An end to end [functional test](https://github.com/Bhamni/emr-functional-tests/tree/master/spec) covering patient registration and consultation goes through multiple apps. We needed to abstract the concept of an app in test code to increase the readability and maintainability. 

The app objects are natural extension of [page objects](https://code.google.com/p/selenium/wiki/PageObjects) recommended for writing functional tests. An app encapsulates all pages and coarse level actions in the app.

<!-- More -->

An simple implementation of app class would be

```ruby registration/app.rb
class Registration::App
    # Pages in this apps    
    def patient_page
        Registration::PatientPage.new
    end

    def visit_details_page
        Registration::VisitDetailsPage.new
    end

    # Coarse level action in this app
    def register_new_patient(options)
        click_link "Create New"
        patient_page.fill(options[:patient]).start_visit(options[:visit_type])
    end
end
```
The test using this simple implementation would look like

```ruby
registration = Registration::App.new
registration.register_new_patient :patient => new_patient, :visit_type => 'OPD'
registration.visit_page.should_be_current_page
registration.visit_page.save_new_patient_visit(visit_info)

clinical = Clinical::App.new
clinical.patient_search_page.should_have_active_patient(new_patient)
clinical.patient_search_page.view_patient(new_patient)
```

The [test code](https://github.com/Bhamni/emr-functional-tests/tree/master/spec) is structured as

<pre>
apps
    registration
        app.rb
        patient_page.rb
        visit_details_page.rb
    clinical
        app.rb
        patient_search_page.rb
features
    new_patient_visit.rb
framework
    app.rb # Base class for other apps
    page.rb # Base class for other pages
</pre>

### Nicer DSL

We are using capybara and Rspec. The lambda syntax, meta programming constructs in ruby and convention based programming allowed us to implement a DSL shown below. The complete code can be found [here](https://github.com/Bhamni/emr-functional-tests/tree/master/spec)

```ruby
feature "new patient visit" do
    background { login('superuser', 'password') }

    scenario "registration and consultation" do
        new_patient = {:given_name => "Ram", :family_name => 'Singh'}

        go_to_app(:registration) do
            register_new_patient(:patient => new_patient, :visit_type => 'OPD')
            visit_page.should_be_current_page
            visit_page.save_new_patient_visit(visit_info)
        end

        go_to_app(:clinical) do
            patient_search_page.should_have_active_patient(new_patient)
            patient_search_page.view_patient(new_patient)
            # ....
        end
    end
end
```

I didn't into details of implementing the DSL. If people are interested, I can write a part 2 of this post.