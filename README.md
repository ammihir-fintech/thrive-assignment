# Thrive-Assignment

## OBJECTIVE:
The goal of this task was to integrate data from three source tables in a SQLite database:
1.users
2.conversation_start
3.conversation_part
---> single unified table called consolidated_messages (named as consolidated_messages_final.csv)

## Query to create the consolidated_messages.csv 

Creates a customer_map CTE to determine the customer user for each conversation thread based on email matches where is_customer = 1
Unifies the opening messages (assigned type 'open') with conversation parts using a UNION ALL
Adds a synthetic part_id = 0 for opening messages and preserves real part_ids from conversation_parts
Assigns a unique row number ordered by conversation_id, created_at to serve as the primary key
Ensures messages are sorted by conversation_id and created_at using ROW_NUMBER() OVER (ORDER BY ...), which also serves as the unique primary key (id) in the consolidated_messages table, as required by the assignment.
Outputs a flat, sorted table named consolidated_messages in the SQLite database

## consolidated_messages_final.csv SCHEMA

The schema for the consolidated_messages is 
id = unique id for each interaction of the thread
user_id = id for the user to whom the thread belongs to 
conversation_id = the id of each conversation thread
email = the email of the message sender
message = the message content
message_type = the type of the message
***part_id = the identifier from parts which will help in normalizing the table.
created_at = timestamp associated with each interaction.


***part_id = for start, 0 is appended while for parts we capture id from the table.

## Data Model

**fact_messages_all**
CREATE TABLE fact_messages_all
(
    id INTEGER  PRIMARY KEY,
    user_id INTEGER ,
    conversation_id INTEGER ,
    part_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (part_id) REFERENCES dim_conversation_parts(part_id)  
    
);

**users (from the original users)**
CREATE TABLE users (
    id INTEGER  PRIMARY KEY, 
    name TEXT, 
    email TEXT, 
    is_customer INTEGER   -- just 1 or 0
);

**dim_conversation_parts**
CREATE TABLE dim_conversation_parts (
    part_id INTEGER PRIMARY KEY,       
    email TEXT ,
    message TEXT,
    message_type TEXT,
    created_at DATETIME
);



## Why dimensional model:
Assignment requires a flattened consolidated_messages table (consolidated_messages_final.csv & also table created in db), consolidated_messages_final is modeled using a fact/dimension schema for extensibility. This separates message events (fact_messages_all) from user info(dim_users) and message content (dim_conversation_parts), enabling scalable analytics.