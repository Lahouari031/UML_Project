# Coach & Track — UML Project

Oct 28, 2026 · GUETARNI Lahouari, NANOUX Augustin, DELPORTE Roman, DEVAUX-RIEVERE Aurélien

## Project

A gym offers group classes and personal coaching. Coaches create training programs and assign them to members. Members book time slots and log their workouts (sets, weights, reps). The app automatically adjusts the weights based on their performance.

To give the state diagrams more substance, the project adds **bookings** (with a waiting list), **memberships** and **automatic progression**.

## Actors

| Actor | Role |
| --- | --- |
| Member | Books time slots, follows their program, logs workouts |
| Coach | Creates and assigns programs, tracks members, runs classes |
| Manager | Manages memberships, time slots and coach accounts |
| System (second option) | Sends reminders, expires memberships, promotes the waiting list |

## Main use cases

The "include" and "extends" relationships are backed by real business rules, not added for show.

**Member**

- Book a time slot: includes "Check active membership", extended by "Join waiting list" if the slot is full
- Cancel a booking: extended by "Apply penalty" if cancelled less than 2 hours before
- View their program
- Log a completed workout
- View their progress (personal records, weight curves)

**Coach**

- Create / edit a program
- Assign a program to a member
- View a member's progress
- Create a group class

**Manager**

- Manage memberships (create, suspend, cancel)
- Manage coaches
- View attendance statistics

The use case diagram is in [`diagrams/use-cases/coach-and-track-use-cases.drawio`](../diagrams/use-cases/coach-and-track-use-cases.drawio) (open it on app.diagrams.net).

## Class diagram

About 12 domain classes, grouped into six blocks.

**Users**

- User (abstract): id, name, email, password
- Member: registrationDate, absenceCount, blockedUntil
- Coach: specialties
- Manager

**Membership**

- Membership: startDate, endDate, state, type (MONTHLY, YEARLY, STUDENT)
- Member 1 — 0..* Membership (history)

**Programming** (the compositions)

- Program: name, goal (MUSCLE_GAIN, STRENGTH, ENDURANCE), state
  - ◆ composed of 1..* WorkoutTemplate (e.g. "Push", "Pull", "Legs")
    - ◆ composed of 1..* PrescribedExercise: sets, targetReps, targetWeight, rest
      - → references an Exercise from the catalogue (aggregation: the exercise exists on its own)
- Coach 1 — 0..* Program (creator)
- Member 0..* — 0..1 Program (active program)

**Exercise catalogue** (justified inheritance)

- Exercise (abstract): name, muscleGroup
  - StrengthExercise: equipment → sets / reps / weight
  - CardioExercise → duration / distance
- Polymorphic method computeVolume(CompletedSet): kg × reps for strength, minutes for cardio

**Logging**

- CompletedWorkout: date, duration, feeling
  - ◆ composed of 1..* CompletedSet: reps, weight, exercise

**Bookings**

- TimeSlot / GroupClass: start, end, maxCapacity, room
- Booking: bookingDate, state, waitingPosition
- Member 1 — 0..* Booking 0..* — 1 TimeSlot (association class)

**Strategy pattern**

- Interface ProgressionStrategy: computeNextWeight(PrescribedExercise, List<CompletedSet>)
  - LinearProgression: all reps completed → +2.5 kg
  - DoubleProgression: increase reps to the top of the range, then the weight
  - Deload: −10% after 2 consecutive failures
- The coach picks the strategy per program: a real pattern you can defend in your oral presentation.

## State diagrams

At least two solid ones: the booking and the membership, plus the program as a bonus.

**Booking**

- Requested → Confirmed (slot available) or Waitlisted (slot full)
- Waitlisted → Confirmed (a spot opens up) or Expired (the class starts)
- Confirmed → Cancelled [> 2 h before] or LateCancelled [≤ 2 h]
- Confirmed → Attended (member showed up) or NoShow
- Business rule: 3 no-shows → member blocked for 7 days

**Membership**

- Active → Suspended (injury, travel) → Active
- Active → Expired (end date passed, "after" time event)
- Active / Suspended → Terminated

**Program** (bonus)

- Draft → Assigned → InProgress → Completed → Archived

## Sequence diagrams

1. **Book a time slot**: Member → UI → BookingService → check membership → check block status → check capacity → alt [slot available] confirm / [full] add to waiting list. Shows alt and opt fragments clearly.
2. **Log a workout**: enter sets → create CompletedWorkout → loop over each exercise → ProgressionStrategy computes the new weight → update the PrescribedExercise. Shows polymorphism in action.
3. **Cancellation with waiting-list promotion** (the most interesting one): the cancellation triggers a notification to the first member on the waiting list.

## Activity diagram

The **member's workout flow**: open today's workout → for each exercise → enter sets → [target reps reached?] → decision → end → compute progression ∥ update personal records (fork/join).

## To add (draft) — gaps vs the syllabus

Rough notes after checking the TI515M/TI615M syllabus. Not final, to sort out with the group.

**Use case documentation**

- Use case description tables: actor, preconditions, main scenario, alternative scenarios, postconditions
- Start with "Book a time slot" and "Log a workout", then the others if time allows
- System sequence diagram (SSD) for those same 2 use cases: actor ↔ system as a black box

**Object diagram**

- One concrete snapshot: member Alex, active Program "PPL Hypertrophy", WorkoutTemplate Push / Pull / Legs, one CompletedWorkout with 3 CompletedSets
- Show links matching the multiplicities of the class diagram

**Class diagram — extra relations**

- Dependency: BookingService ⇢ NotificationService
- N-ary association? Maybe Member – Coach – Program (who assigned what to whom). Only if it makes sense, don't force it
- Stereotypes: interface, abstract, enumeration (BookingState, MembershipState...)

**Sequence diagrams — what the course expects**

- MVC stereotypes on lifelines: BookingUI (view) → BookingController (controller) → Booking / TimeSlot (model)
- Add a par fragment: on cancellation, notify the waitlisted member in parallel with updating the slot
- Current operators: alt, opt, loop. With par → all 4 covered

**Communication diagram**

- Convert "Book a time slot" from sequence to communication (numbered messages 1, 1.1, 1.2...)
- Good for a slide comparing the two views

**Activity diagram — advanced concepts**

- Swimlanes: Member / System / Coach
- Object flow: CompletedWorkout passed to the progression step
- Signal: "spot freed" signal triggering the waiting list promotion
- Exception: membership expired during booking

**Design patterns from the course**

- Observer: waitlisted members observe the TimeSlot, notified when a spot frees up (textbook case)
- Factory: ExerciseFactory creates StrengthExercise or CardioExercise from a type
- Keep Strategy (progression) as the original one
- Maybe Singleton for the repository / app context? To discuss, can look lazy

**Organisation**

- Same class and method names in every diagram and in the Java code (graded)
- Visual Paradigm: try code generation + reverse engineering
- Implementation = bonus only (max 3 pts with patterns and extra diagrams) → diagrams first
- Groups of 2 or 3, at least 2 follow-up sessions, oral at the last session

**Open questions**

- Is coding mandatory for our lecturer, or just bonus?
- Do state / component / deployment diagrams count as "other diagrams" bonus? Ask at the follow-up

## Payment & subscription system (draft)

Rough brainstorm. Idea: Membership becomes a real Subscription to a Plan, paid through an external payment provider, renewed automatically.

**New actor**

- Payment Gateway (external system, secondary actor): authorises, captures and refunds payments
- Manager gets new use cases (plans, refunds, payment reports)

**New use cases**

- Member: Subscribe to a plan (includes "Pay", extended by "Apply promo code")
- Member: Start a free trial (7 days?)
- Member: Change plan (upgrade / downgrade, extended by "Compute prorated amount")
- Member: Cancel subscription (effective at the end of the current period)
- Member: Update payment method
- Member: View payment history / download invoices
- System: Renew subscription automatically (includes "Pay", extended by "Retry failed payment")
- System: Send payment reminder / failure notification
- Manager: Create / edit plans and prices
- Manager: Issue a refund
- Manager: View revenue and unpaid subscriptions
- Link with existing: "Book a time slot" already includes "Check active membership" → becomes "Check active subscription"

**New / changed classes**

- Plan: name, price, billingPeriod (MONTHLY, QUARTERLY, YEARLY), classesPerWeek, includesCoaching, trialDays
  - maybe StudentPlan with a discount + proof of student status
- Subscription (replaces Membership): startDate, currentPeriodEnd, nextBillingDate, state, autoRenew
  - Member 1 — 0..* Subscription (history), Subscription * — 1 Plan
- Payment: amount, date, status, failureReason, attemptNumber
  - Subscription 1 — 0..* Payment
- Invoice: number, issueDate, amountExclTax, VAT, total
  - Payment 1 — 0..1 Invoice
- PaymentMethod (abstract): token, isDefault, expiryDate
  - CardPayment, DirectDebit (SEPA), maybe Wallet
  - Member 1 — 1..* PaymentMethod
- PromoCode: code, percentOff, validUntil, maxUses
- Refund: amount, reason, date (Payment 1 — 0..* Refund, partial refunds possible)
- Security note: never store the card number, only the token returned by the gateway

**State diagrams**

- Subscription: Trial → Active → PastDue (payment failed) → Active (retry OK) or Suspended (3 retries failed) → Cancelled
  - Active → PendingCancellation (member cancels) → Cancelled at period end
  - Active → Active on renewal (self-transition), Active → Expired if autoRenew off
  - Suspended → Active when the member pays the overdue amount
- Payment: Pending → Authorised → Captured → (Refunded / PartiallyRefunded)
  - Pending → Failed (declined card, insufficient funds)

**Sequence diagrams**

- Subscribe to a plan: Member → SubscriptionUI → SubscriptionController → PromoCode check → PaymentGateway.authorise → alt [OK] create Subscription + Invoice / [declined] show error
- Automatic renewal: Scheduler → SubscriptionController → loop (max 3 attempts) PaymentGateway.charge → alt success / failure → par notify member ∥ update state to PastDue

**Activity diagram**

- Monthly renewal process with swimlanes System / Payment Gateway / Member
- Retries at D+1, D+3, D+7 → after 3 failures: Suspended, bookings blocked
- Fork: generate invoice ∥ send confirmation email

**Business rules**

- Upgrade: pay the prorated difference immediately; downgrade: applied at next period
- Cancellation: access kept until currentPeriodEnd, no refund
- PastDue: member can still book for 7 days, then blocked
- One promo code per subscription
- Refund only by the Manager, within 14 days

**Design patterns that fit naturally**

- Adapter (from the course): PaymentGatewayAdapter wraps the external provider's API behind our own PaymentGateway interface → can switch provider without touching the rest
- Observer: payment events (succeeded, failed, refunded) notify Subscription, NotificationService, InvoiceService
- Factory: PaymentMethodFactory creates CardPayment or DirectDebit
- Command (from the course): Charge, Refund as command objects → logged, retryable
- Strategy (already used): PricingStrategy for normal / student / promo price?

## Group classes with a weekly quota (draft)

Idea: the subscription gives a number of coach-led group classes per week (e.g. 3), and the app manages that quota.

**Plans (examples)**

- Basic: gym access + 1 class / week
- Standard: gym access + 3 classes / week
- Premium: unlimited classes

**Classes**

- Plan: weeklyClassQuota (null or -1 = unlimited)
- WeeklyQuota: weekStart, allowed, used
  - Subscription 1 — 0..* WeeklyQuota (one per week, history)
  - methods: remaining(), canBook(), consume(), giveBack()
- GroupClass: coach, title, start, end, maxCapacity, room
  - Coach 1 — 0..* GroupClass
- Booking stays the link Member — GroupClass, and now also uses a credit from WeeklyQuota

**Business rules**

- Booking a class uses 1 credit of the week of the class (not the week of the booking)
- Quota reached → booking refused, or buy an extra class (single payment, links to the payment system)
- Cancel more than 2 h before → credit given back
- Late cancel (less than 2 h) or no-show → credit lost
- Waitlisted = no credit used; credit used only when the spot is confirmed
- Unused credits are lost at the end of the week (reset every Monday)
- Upgrade mid-week → new quota applies right away; downgrade → next week
- Subscription PastDue or Suspended → no new bookings

**Use cases**

- Book a class: includes "Check active subscription" and "Check remaining credits", extended by "Buy an extra class" when the quota is reached
- View my remaining classes this week
- Coach: create a group class, view who is booked, mark attendance (→ Attended / NoShow)

**Sequence: book a class**

- Member → BookingUI → BookingController → Subscription.isActive() → WeeklyQuota.canBook() → GroupClass.hasSpot()
- alt [credit + spot] create Booking, consume() / [credit, no spot] waiting list / [no credit] offer extra class → PaymentGateway

**State diagram addition (Booking)**

- Confirmed → Cancelled [> 2 h]: giveBack()
- Confirmed → LateCancelled or NoShow: credit stays used

**Design pattern**

- Strategy for the quota: QuotaPolicy interface with LimitedWeeklyQuota and UnlimitedQuota → the plan decides, no if/else everywhere
