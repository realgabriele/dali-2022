# DALI project
## by Gabriele Tagliente, matr. 279677

---

The general idea for this project is to implement the job seeking proceess.  
In this model, Businesses offer job positions,
while Workers offer their work.  
This is done with the help of a Broker (agenzia iterinale),
which notifies Workers/Busenesses for new job/work,
and registers their interests.


# TOC
1. Class Diagram
1. Sequence Diagram
1. Code
1. Installation
1. Example


# Class Diagram
![Class Diagram Image](Images/class_diagram.drawio.png)

Components:
* **Worker**: this class represents a worker, which has some years of experience and wants a minimum salary from the job to accept it.  
All the workers are extensions of this general class.
* **Business**: this class represents a business, which offers some pay for the job and wants a minimum amount of years of experience for a woirker to hire it.  
All the businesses are extensions of this general class.
* **Broker**: the agency which collects all the data, put workers and businesses in contact, and registers mutual interests.
In the end takes care of making new job contracts.  
_Workers_ and _Businesses_ only talk via the _Broker_


# Sequence Diagram
![Sequence Diagram Image](Images/sequence_diagram.drawio.png)

This diagram shows how the Broker registers new availabilities
and notifies the Workers/Businesses  
(in this model only one for each type is shown,
but the availability notification will be broadcasted to all
agent of the same type).

After a mutual interest is shown between a Worker and a Business,
the Broker will notify both ends for the new contract.

# Code
Following there are reported some snippets of the code,
analyzing the most interesting parts.

## Subscribe
On startup, each agent will subscribe to the *Broker*,
indicating the group the are part of (*Business* or *Worker*).

On the *Broker* side, this is the code that handles the request:
```prolog
:- dynamic type/2.

subscribeE(Agent,Type) :> assert(type(Agent,Type)).
```
Allowing a dynamic fact, we can assert new types
and store them in the Knowledge Base.

## Offer
When the agents receive the `go` event,
they will make an offer to the *Broker*.
```prolog
experience(5).

goE :> experience(X), messageA(broker, send_message(work_offer(Me, X),Me)).
```
This is made consulting dynamically the knowledge base
and presenting their offer (either in pay or experience)

On the other hand, the *Broker* will receive the offer
and broadcast it to all the agents of the other type.  
Thanks to the previous `type` statement,
it is now possible to send a message to all agents
of the same type:
```prolog
% Create list of agents of type Type
all_agents(Type,Input,Input) :- \+ (type(Ag,Type), \+ memberchk(Ag,Input)).
all_agents(Type,Input,Output) :- type(Ag,Type), \+ memberchk(Ag,Input), !, all_agents(Type,[Ag|Input],Output).

% send a message to all Business agents
message_businessesA(Worker,Exp) :> all_agents(business,[],Businesses), message_workA(Businesses,Worker,Exp).

% loop the list to send each one the message
message_workA([],X,Y).
message_workA([E|L],Worker,Exp) :>
	messageA(E,send_message(work_available(Worker, Exp),Me)),
	message_workA(L,Worker,Exp).
```

## Interest
Upon offer reception,
an agent can decide wheter the offer is interesting for them.
In this case they use the fact in the knowledge base
to check if it satisfies the minimum requirements:
```prolog
min_experience(5).

% External Event: offer received
work_availableE(Worker, Exp) :> worker_interestA(Worker, Exp).

% The following are the preconditions and the action itself.
worker_interest(Worker, Exp) :< min_experience(X), X =< Exp.
worker_interestI(Worker, Exp) :> messageA(broker,send_message(interest(Me, Worker),Me)).
```

On the other end, the *Broker* will take care of registering the mutual interests:
```prolog
:- dynamic interest/2.

interestE(From,To) :> assert(interest(From,To)).
```

## Match
Each time a change is made in the KB, the *Broker* will check
wheter a new *match* has been created.  
In case of positive result, it will make a new contract
and it takes care of notify the parts.
```prolog
match(A,B) :- interest(A,B), interest(B,A), type(A,worker), type(B,business).
matchI(A,B) :> messageA(A, send_message(new_contract(B),Me)),       
    messageA(B, send_message(new_contract(A),Me)),
	retract(interest(A,B)), retract(interest(B,A)).
```


# Installation
Installation instructions:
no particular information is required in order to install this project,
apart from the normal [DALI](https://github.com/AAAI-DISIM-UnivAQ/DALI) installation instructions.

The project has been implemented and executed on a Linux OS.

* Have a running [Sicstus](https://sicstus.sics.se/) installation.  
    See [this guide](https://sicstus.sics.se/eval.html) to seek the product evaluation
* open a terminal in the directory where you want to install it.
* clone this repository on the local system:  
    `git clone https://github.com/realgabriele/dali-2022.git`
* enter into the repo directory:  
    `cd dali-2022`
* open the file `src/startmas.sh` to edit the `SICSTUS_HOME` variable and adjust it to your own sicstus installation path.

### Program Execution
* start the program:  
    `./startmas.sh`
* use the commands in `commands.txt` in the *active user* window.

### Exit the program
* To interrupt the entire execution,
press any key on the original terminal.

### Common problems
* After a single execution,
it is needed to wait for 60 seconds
before restart another round.


# Example
Following some screenshot showing an example of execution.

ToDo: screenshot

