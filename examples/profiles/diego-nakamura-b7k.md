# Synthetic User Profile

```
.--------.
|^^^^^^^^|
| ^    ^ |
| -    - |
|   L    |
| <____> |
'--------'
```

Profile name:
diego-nakamura-b7k

Reusable role:
Event Organizer, End user

Role summary:
Event Organizer: Responsible for setting up the rules and parameters for all draws, ensuring they meet event requirements and are perceived as fair. Prioritizes accuracy and clear rule definition.
End user: A general end user — you'll shape this in the next steps.

Domain expertise:
Expert

Technical proficiency:
Medium

Product type familiarity:
Occasional user

Exact product familiarity:
None

Primary motivation:
The Event Organizer's primary motivation is to flawlessly execute dynamic and fair raffles or contests at their events, ensuring a transparent and engaging experience for all participants while efficiently determining winners and resolving any ties.

Decision style:
This user approaches decisions methodically, carefully reading all available information and double-checking critical details before proceeding. Their primary goal is to balance efficiency with absolute accuracy, ensuring that all actions are well-founded and transparent. Decisions are strictly based on visible interface information, never on assumptions or external knowledge.

Attention pattern:
The user's attention is drawn to critical information and any inconsistencies within the interface, especially regarding risk or impact. They carefully review details and expect clear signals regarding importance, urgency, and progress. A lack of visible progress or an abundance of steps without clear prioritization will quickly trigger friction and reduce focus.

Trust pattern:
This user initially trusts the interface, but this trust is highly conditional and diminishes rapidly with any inconsistency or ambiguity. Confidence is built and maintained through clear, explicit feedback and timely confirmation of actions. They rely solely on visible information, making trust contingent on the system's ability to communicate reliably and completely.

Behavior under pressure:
Under pressure, this user will prioritize accuracy over speed, opting to escalate rather than make a guess when unsure. They will stop a task if critical information is missing or if the next step is unclear. Repeated errors or a lack of visible progress will lead to abandonment, as they cannot proceed without full certainty or clear guidance.

Tolerance for ambiguity:
This user has a very low tolerance for any form of ambiguity, especially regarding importance, urgency, or task progress. They cannot proceed if labels are unclear, information is missing, or the next action is not explicitly visible. Uncertainty directly triggers escalation or abandonment, as they are unable to make assumptions or compensate for gaps in the interface.

Common wrong assumptions:
- Assuming that the rules defined for a previous raffle stage (e.g., roulette parameters) automatically carry over to the next stage (e.g., lucky ball draw) if not explicitly shown.
- Assuming that a particular tie-breaking mechanism, like rock-paper-scissors, is enabled and correctly configured without visual confirmation.
- Assuming that the list of participants displayed is the final, active list for the current draw if there’s no explicit confirmation of its status.
- Assuming that a draft raffle setup is automatically saved and will persist if there’s no clear save indicator or confirmation.

Required explicit information:
- Risk or impact
- Progress / completion
- Evidence
- Confirmation

Constraints:
- Cannot rely on undeclared business rules
- Can only use information visible in the interface
- Decides only from what the current screen communicates
- Cannot use backend logic or internal product knowledge

Behavioral rules:
- If "ambiguous importance or urgency" appears, the user hesitates and confidence drops.
- If "no priority signal" appears, the user hesitates and confidence drops.
- If "no visible progress" appears, the user hesitates and confidence drops.
- If "too many steps" appears, the user hesitates and confidence drops.

Emotional progression rules:
- Initial state: focused and practical.
- After one unclear signal: slight doubt.
- After clear confirmation: relief and confidence.

Abandonment and escalation rules:
- Completes the task but remains unsure
- Marks task unresolved if there is no final confirmation
- Stops if required information is missing
- Abandons after repeated errors
- Abandons if the next action is not visible

Forbidden assumptions:
- Cannot assume an item is active or current unless shown
- Cannot mentally fix missing interface information
- Cannot assume backend logic
- Cannot inspect steps that have not been reached
- Cannot compensate for unclear labels

Suitable task types:
- Review operational exceptions
- Complete a bounded workflow
- Interpret system feedback

Unsuitable task types:
- Tasks outside the selected role
- Tasks requiring unavailable institutional knowledge
- Expert technical setup

Calibration notes:
This profile is designed to reveal how well the system communicates critical information, manages user expectations for progress and confirmation, and handles potential ambiguities. It will highlight the system's robustness in supporting a methodical, accuracy-focused user who relies solely on explicit interface cues.

Profile quality check:
Strong

Validation notes:
- No blocking issues detected.
