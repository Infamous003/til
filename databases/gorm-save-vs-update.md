# GORM Save() vs Updates()

Save() either updates an existing record or inserts a new one. It does so by seeing if the incoming struct's PK(ID) field is populated or not. If ID is null, then it inserts a new record, else, it updates an existing record. 

- Save is can be used for updates, but in cases where you do not have access to PK, Save can end up inserting a new record instead of Updating. You can compensate this by fetching before upadate, but it's an extra query to DB

- This is where Updates() comes in. It partially upates fields that you send. It is generally more suitable for updating as you specify what row should be updates using Where() clause

This bites you especially when the "ID" you have on hand is a
foreign key on the target table, not its actual primary key —
e.g. you have `userID`, but `user_profiles` has its own `id` column
with `user_id` just referencing it. A struct built from `userID`
alone has no real PK set, so Save() will insert, not update.
