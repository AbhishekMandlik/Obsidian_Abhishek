**Top Tier: Must Know (90% of LLD Interviews)**

If an interviewer asks an LLD question (like _"Design a Parking Lot"_ or _"Design a Vending Machine"_), you will almost certainly need to write out one of these:

- **Strategy Pattern:** **Extremely common.** Used whenever you have multiple algorithms for a single action.
    - _Interview context:_ Switching payment options (PayPal vs. Credit Card) or vehicle ticket pricing calculations.
- **Observer Pattern:** **Extremely common.** Used for event-driven systems.
    - _Interview context:_ Building a real-time notification engine (Email/SMS/Push) or a live stock price ticker.
- **State Pattern:** **Highly common.** Used when an object's behaviour changes completely based on its current operational state.
    - _Interview context:_ This is the exact solution required to crack the classic _Vending Machine_ or _Elevator_ interview questions.
- **Factory / Factory Method:** **Highly common.** Used to hide raw instantiation logic.
    - _Interview context:_ Safely generating different types of objects (e.g., creating domestic vs. international cargo shipping objects) without using the `new` keyword everywhere.

---

**🥈 Second Tier: Good to Know (Asked 30% of the time)**

These are occasionally tested as follow-up questions to see how you optimize your code or handle specific edge cases.

- **Singleton Pattern:** Mostly asked as a conceptual multithreading question. You need to know how to write a **Thread-Safe, Double-Checked Locking Singleton**.
- **Decorator Pattern:** Used when you need to attach optional responsibilities to an object dynamically without modifying the core class.
    - _Interview context:_ Adding custom toppings in a _Pizza Ordering System_ or features in a _Coffee Maker Simulator_.
- **Builder Pattern:** Mentioning this pattern shows production experience. It is helpful when initializing massive entity classes that contain dozens of optional fields.

---