# Interviewer: Talk about your understanding of MySQL MVCC. I said I knew about MVCC, but the interviewer told me to go back and wait for news.

**Original** **Fox** [Fox爱分享](javascript:void(0);)

 *October 7, 2025, 06:00*

**Scene Restore **

**Interviewer:“ **Talk about your understanding of MySQL's MVCC。 **” **

**Me:“MVCC is multiversion concurrency control, used to achieve read committed and repeatable read isolation levels. It is implemented by version chain and ReadView, read does not block write, write does not block read.” **

**Interviewer:"Well, that's good. **Can you elaborate on how ReadView determines data visibility？ At the reusable level, when was it generated? **” **

**Me: (with a panic) "ReadView is to determine whether the version of the data can be seen. Is the transaction generated at the beginning?" **

**Interviewer:“ **undo logWhat does it do in the MVCC? If a piece of data is updated frequently and the version chain is long, will it affect query performance? **” **

**Me: (Starting to Shout) "Undolog is used to roll back... The version chain is long and should... Will it be a little slower?" **

**Interviewer:“Okay, i see. Today's interview comes here first. We'll let you know if there's news.” **

**Result : **No doubt , go back and wait for the news = interview hang .

#### **1. Depth analysis: three elements of the core mechanism of MVCC**

**The interrogator's questioning points directly to the core mechanism of MVCC, and it is far from enough to know the concept. To truly understand MVCC, you must grasp its three core elements, which act like a sophisticated collaborative system that together ensure data visibility at high concurrency. **

**1. Hidden field: "Identity information" of the data **

**Every record stored in InnoDB contains several hidden fields invisible to the user, which are the "data base" of MVCC: **

* `<span leaf="" class="js_darkmode__text__40"><span textstyle="" class="js_darkmode__text__41">DB_TRX_ID(6字节)</span></span>`  **TransactionID ** **。The transaction ID that last** **inserted **or **updated **this record。 The delete operation is also treated as an update inside InnoDB （setting the delete flag）.
* `<span leaf="" class="js_darkmode__text__54"><span textstyle="" class="js_darkmode__text__55">DB_ROLL_PTR(7字节)</span></span>`  **Roll back the pointer ** **。Points to the** **previous version of **the data in the undo log。Multiple versions are strung together into a "version chain" through this pointer.
* `<span leaf="" class="js_darkmode__text__65"><span textstyle="" class="js_darkmode__text__66">DB_ROW_ID(6字节)</span></span>`  **Okay。ID ** **。If the table does not define a primary key, InnoDB automatically generates a hidden primary key clustered index using this field.**

**2. UndoLog: The Time Tunnel of Data **

Undo Log is used not only for **transaction rollback **, but also for MVCC to achieve **multiple versions **of the key。It stores the original information before the data is modified.

* **The formation of a version chain****When you execute** `<span leaf="" class="js_darkmode__text__87">UPDATE</span>`Statement, InnoDB first stores a copy of the current data (before modification) into the undo log, then overwrites the current row with the new data, and updates the `<span leaf="" class="js_darkmode__text__90">DB_TRX_ID</span>`and `<span leaf="" class="js_darkmode__text__93">DB_ROLL_PTR</span>`The rollback pointer points to the old version record that was just stored in the undo log. In this way, through the rollback pointer of the current record, we can find all the historical versions of it, forming a chain table.
* **Effect****When a transaction needs to read data, if the current version is not visible to the transaction, MVCC moves along the version chain, looking for historical snapshots that match its visibility.**

**3. ReadView: "Visibility Rules" for Transactions **

**A read view is a transaction that performs a ** **snapshot read ** **（ normal **`<span leaf="" class="js_darkmode__text__110">SELECT</span>`Statement （s） ） to produce a  **consistent view ** **。It defines a criterion of judgment to determine which version of the data the current transaction will see.ReadView consists of four key components: **

* `<span leaf="" class="js_darkmode__text__118"><span textstyle="" class="js_darkmode__text__119">m_ids</span></span>`**At the moment the ReadView is generated , all**  **active ** **（ non-committed ） transaction IDs in the system .**
* `<span leaf="" class="js_darkmode__text__127"><span textstyle="" class="js_darkmode__text__128">min_trx_id</span></span>``<span leaf="" class="js_darkmode__text__131">m_ids</span>`The minimum value in the collection.
* `<span leaf="" class="js_darkmode__text__135"><span textstyle="" class="js_darkmode__text__136">max_trx_id</span></span>`**When a ReadView is generated, the system should assign** **next transaction **ID value。（Note: `<span leaf="" class="js_darkmode__text__143">max_trx_id</span>`It's not. `<span leaf="" class="js_darkmode__text__146">m_ids</span>`The maximum value in.
* `<span leaf="" class="js_darkmode__text__150"><span textstyle="" class="js_darkmode__text__151">creator_trx_id</span></span>`**The transaction that generates this ReadView has its own ID.**

**An important test! : The timing of ReadView generation determines the isolation level **

* **Read Committed (Read Committed, RC)** **Each **time a snapshot read is performed a new ReadView is created .
* **Repeatable Read (RR)** **Only when a transaction** **first **performs a snapshot read is a ReadView generated, and all subsequent read operations **reuse **this ReadView。This is the root of "repeatable read" under RR level

#### **II. Visibility judgment algorithm: the "soul" of MVCC**

**With the version chain (data) and ReadView (rules), the judgment process becomes rules-based. **

**How do I tell if a version in the version chain is visible to the current transaction? **

**The algorithm is as follows: **

1. **If this version of the data is available** `<span leaf="" class="js_darkmode__text__193">DB_TRX_ID</span>` **Be equal to ** `<span leaf="" class="js_darkmode__text__197">creator_trx_id</span>`, indicating that this is data modified by the current transaction itself ,  **visible ** **.**
2. **If this version of the data is available** `<span leaf="" class="js_darkmode__text__206">DB_TRX_ID</span>` **Less than ** `<span leaf="" class="js_darkmode__text__210">min_trx_id</span>`, indicating that this version was committed before the ReadView was created ,  **visible ** **.**
3. **If this version of the data is available** `<span leaf="" class="js_darkmode__text__219">DB_TRX_ID</span>` **More than equals ** `<span leaf="" class="js_darkmode__text__223">max_trx_id</span>`, indicating that this version was modified by a transaction that started after the ReadView was created ,  **not visible ** **.**
4. **If this version of the data is available** `<span leaf="" class="js_darkmode__text__232">DB_TRX_ID</span>`in `<span leaf="" class="js_darkmode__text__235">[min_trx_id, max_trx_id)</span>`In a range, the check is performed `<span leaf="" class="js_darkmode__text__238">DB_TRX_ID</span>`Are you here? `<span leaf="" class="js_darkmode__text__241">m_ids</span>`In the list:

   * **in****When the ReadView is created , the transaction that modified this data version is still active （ uncommitted ） , so the version is**  **not visible ** **.**
   * **not to be present****When the ReadView is generated , the transaction that modified this data version has been committed , so the version is**  **visible ** **.**

**If the current version is not visible to the transaction, follow the version chain **`<span leaf="" class="js_darkmode__text__263">DB_ROLL_PTR</span>`Find the previous version ,  **repeat the above judgment process ** **, until you find a version that meets the criteria . **

 **Performance considerations ** **: A long version chain does affect retrieval efficiency, because the chain must be traversed。But it's a trade-off ****between space （multiple versions） and time （read-write concurrency） **trade-offs。In practice, the version chain does not grow indefinitely because of the purge mechanism of the undo log.

#### **III. Standard Answer Template for Interviews (High Version)**

**“My understanding of MVCC mainly has the following aspects: **

1. **First of all , the core purpose of MVCC is to achieve non-blocking read in a lock-free way , which greatly improves the concurrency performance of the database . **Its implementation relies on three core mechanisms : hidden fields （ `<span leaf="" class="js_darkmode__text__288">DB_TRX_ID</span>`, `<span leaf="" class="js_darkmode__text__291">DB_ROLL_PTR</span>`), UndoLog version chain, and ReadView.
2. **Secondly , its working principle is data multi-version + visibility judgment . **InnoDB maintains a chain of historical versions of data through undo logs。When a snapshot read is performed, the transaction generates a ReadView based on the isolation level.
3. **Visibility is the key . **The ReadView defines which data versions are visible to the current transaction。The judgment rule is:
   * **Visible if the data transaction ID is less than the minimum active transaction ID in ReadView.**
   * **Is not visible if it is greater than or equal to the next transaction ID to be allocated.**
   * **If it is somewhere in between, it depends whether it is in the active transaction list, visible when it is not and invisible when it does not. This process recursively finds along the version chain.**
4. **When it comes to isolation levels, the timing of ReadView generation is the key difference. **

* **At** **the RC **level , a new ReadView is generated for each read , resulting in each read being able to read the committed new data and avoiding dirty reads , but not repeatable reads .
* **At** **RR **level , only the first read generates a ReadView and is reused to achieve repeatable read .

1. **Finally , MVCC solves phantom reads with Next-Key Lock . **MVCC itself solves snapshot phantom reads by snapshot reads , and current reads （ such as `<span leaf="" class="js_darkmode__text__336">SELECT ... FOR UPDATE</span>`The phantom reading below () needs to be solved by a gap lock.

**In summary, MVCC is the cornerstone of InnoDB's implementation of high-performance transaction isolation, and understanding the version chain and ReadView is the essence of it." **

**summary **

**The next time you interview, don't just throw out a name。From ** **purpose, core composition, working principle （algorithm）, differences under different isolation levels and classic problems to solve ** **, you can give a "standard answer" that makes the interviewer shine. **

**If you think it is useful ,**  **give a like and save it ** **, maybe you will use it in the next interview**

**If you want to know more high-frequency interview questions , please pay attention to the WeChat public number "** **Fox Love Sharing** **" to get the million-word interview bible .**
