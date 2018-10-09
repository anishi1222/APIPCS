# Demo/Training Environment

PaaS PMs can use PaaSProdUCM to demo API Platform.  You will need to have an approved account on this domain which is available to PaaS PMs only.

The two demo environments are:
- APICVCDemo 
- APIPlayGround

**You do not have full admin rights to this environment.**  Not all tutorials will work here.  For example, a tutorial involving grants may run into conflicts given the shared nature of this environment.

When you use this environment, you need to be aware that others may be creating objects using the same credentials.  This environment is intended for you to be able to quickly try and demo the product.  It should not be used for deep-level admin/granting work.  If you use this environment, we highly recommend you add identifying elements to all of the objects you create in a tutorial.  For example, if the tutorial tells you to create an API named "TicketService", you might try creating "TicketService+your initials" (e.g. TicketServiceRW). 

When you create a request policy, you need to make sure you differentiate it.  When you deploy to a gateway, the end-point must be unique.  We suggest simply adding a date and your initials.  So for example if a tutorial suggests /ticketservice, you should use /`<date>`/`<your initials>`/ticketservice  (e.g. /180118/rw/ticketservice).  Yes, we know this is long, but it will help you to have a unique end-point so you can play nice with others.  **Remember, this is just needed for the shared PM instance.**  If you are using a VM or GSE, you would not have to differentiate your APIs in order to maintain separation from others using the same account.
