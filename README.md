# Solana Journal

## 🎬 Recorded Sessions
| Link | Instructor | Event |
| ---- | ---------- | ----- |
| [<img src="https://raw.githubusercontent.com/Solana-Workshops/.github/main/.docs/youtube-icon.png" alt="youtube" width="20" align="center"/> Recording](https://youtu.be/0RW4V7-a5FU) | Joe Caulfield | N/A |

## 📗 Learn

This workshop demonstrates creating a journal using on-chain data on Solana.   
    
This program makes use of specialized accounts on Solana called Program-Derived Addresses (PDAs).

### About PDAs

A PDA is an account on Solana whose public key is derived using a hash algorithm from the program's address. For this reason, these accounts don't have private keys, so they don't require any client-side signatures to perform actions.   
   
The way this works, at a high-level, is as follows:
* The account's public key is derived using a list of "seeds" - which are variables serialized into bytes
* The program can then use these seeds to algorithmically authorize changes to that account, rather than requiring a client-side private key signature
* On the client-side, as long as anybody knows the seeds that were used to create this account, they can derive the account's address in a deterministic fashion
   
PDAs allow you to expand on the data capabilities of a Solana program without burdening your users with unnecessary signatures. 

You can find more information on PDAs in the [Solana Docs](https://docs.solana.com/developing/programming-model/calling-between-programs#program-derived-addresses), [Solana Cookbook](https://solanacookbook.com/core-concepts/pdas.html#facts), or this [Solana Bytes tutorial video](https://www.youtube.com/watch?v=ZwFNPvqUclM&list=PLilwLeBwGuK51Ji870apdb88dnBr1Xqhm&index=8).

### Creating a Journal

In this demo, we are going to use a PDA to create a journal for a user.   
   
In this example, one user can only have one journal, so our mapping of users to journals is 1 to 1.   
   
We invoke the System Program to create this account using `invoke_signed` - which will allow us to create this account using only it's seeds, rather than a private key signature.   
Our program also owns this account, so it can modify it's data.

https://github.com/Solana-Workshops/solana-journal/blob/main/journal-program/src/instructions/init_journal.rs#L52-L74

Once we've created the journal, we can derive it's address using only the seed prefix ("journal") and the user's public key - which we know because they're connected to our app!

https://github.com/Solana-Workshops/solana-journal/blob/main/app/src/stores/useJournalStore.tsx#L20-L26

### Creating Entries

Once we've created a journal for our user, now we can start to write new entries into this journal!   
   
Our instruction is actually going to leverage the count of entries stored in the Journal account to know which number this entry represents.   
   
This is because we can have multiple entries for one journal, so a 1 to 1 mapping of seeds won't work. We have to involve a **counter**.

You can see first we load the corresponding journal that this entry will belong to, and check it's value for number of entries:
https://github.com/Solana-Workshops/solana-journal/blob/main/journal-program/src/instructions/new_entry.rs#L45-L58

Then we can use that information to create the data for the new entry:
https://github.com/Solana-Workshops/solana-journal/blob/main/journal-program/src/instructions/new_entry.rs#L52-L58

Then we create the new entry account with `invoke_signed` like we saw with the Journal:
https://github.com/Solana-Workshops/solana-journal/blob/main/journal-program/src/instructions/new_entry.rs#L65-L82

And finally, we can serialize that entry data into the entry account, and also update our Journal's number of entries to make sure it reflects the current count:
https://github.com/Solana-Workshops/solana-journal/blob/main/journal-program/src/instructions/new_entry.rs#L84-L95

On our client side, we can leverage that count of entries stored within the Journal account to iteratively load all of the entries!
https://github.com/Solana-Workshops/solana-journal/blob/main/app/src/stores/useEntryStore.tsx#L21-L47