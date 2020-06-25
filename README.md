DSL for Gherkins

Feature file:
```
Feature: Home page

Background:
  Given a user with a valid id
  And the account service is available

Scenario: A user pushes home button
  Given the user is on the home page
  When the user pushes the button
  Then the user sees a confirmation that the button was pressed
```

The Gherkin spec defines a list of valid keywords that are defined in feature files. These can be mapped to keywords in an EDN file:

* Feature
* Scenario
* Given, When, Then, And, But

As data:
```
(def gherkins
  {:feature
    {:description "Home page"
     :definition {:background [{:given "a user with a valid id"}
                               {:and "the account service is available"}]
                  :scenarios [{:scenario "A user pushes the home button"
                               :steps [{:given "the user is on the home page"}
                                       {:when "the user pushes the button"}
                                       {:then "the user sees a confirmation that the button as pressed"}]}]}}})
```

Adding handlers in Clojure:

In order for the Gherkins to be executed, each step must be matched with the `:step-type` and the `:text`. So as an example, below the handler would be matched with the data definition of the Gherkin:
```
(def valid-id-handler
  {:step-type :given
   :text "a user with a valid id"
   :handler-fn (fn [context] (assoc-in context :user-id "00000-0000-0000"))})
```

Matching with the data definition of the Gherkins:
```
(defn handler-key [handler]
  {(:step-type handler) (:text handler)}) 

;; Picture having a set of every possible step definition in the schema
;; extracted, e.g. #{{:given "the user is on the home page"} {:when "the user pushes the button"} ...}
(defn attach-handlers [gherkins handlers]
  (let [steps (extracted-steps gherkins)]
    (for [s steps]
      (let [handler (filter #(= s (handler-key %)))]
        (assoc-in s :handler handler)))))
```


Executing step definitions in order:
As an idea, I picture each step definition building up a state. The background definition of the Gherkin would create an init state that will be used by all scenarios of the feature.
The state would be built up from the background and to the scenario's `given` and `when` steps, then the `then` steps would take that constructed state and apply asserts to figure out whether the state was properly constructed.
```
(let [context (or background-context {})]
  (loop [step steps]
    (recur (rest steps) ((-> step :handler :handler-fn) context))))
```
