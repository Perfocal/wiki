# Admin

-----

# Admin - Promocodes

## View

New promocode list screen displays a list of all promocodes in the database.
![image](https://user-images.githubusercontent.com/15820577/41857083-8da76d3a-788e-11e8-95bc-f0602e398ffd.png)

## Deletion

It is possible to select a number of promocodes and delete them in batches, or one-by-one, by checking the checkbox on the left-hand side of the promocode item's row.

### Note: **THIS IS NOT REVERSABLE**

## Creation

There is a new feature to create new promocodes (via the button at the top of the list) and edit existing promocodes (via the 'Edit' button on the right-hand side of the promocode item's row).
![image](https://user-images.githubusercontent.com/15820577/41857635-e6b1886a-788f-11e8-9a20-fbb239231453.png)

A modal is brought up to display a set of inputs to modify and set the attributes of the promocode.
![image](https://user-images.githubusercontent.com/15820577/41857740-1f363c1c-7890-11e8-83f4-6a794f90adc1.png)

The `name`, `amount`, and `type` are required fields while the rest are optional and only change the validation settings of the promocode.
Pressing 'Save' will save the promocode to the database, close the modal and fetch a new promocode list from the server.
Pressing 'Cancel' will remove any changes made and close the modal.

Batch creation is as follows:

1. Admin presses 'Create New Batch' button
2. Modal is opened
3. Admin inserts the list of names into the multiline input box separated by either `,` or `,[space]`
4. Names are split by `,`, have any whitespace trimmed and are added to an array
5. Admin fills in other promocode data and presses 'Save'
6. Promocodes are generated one-by-one from the list of names.

### Note: The names are 'WYSIWYG' (what you see is what you get), i.e. if you enter `test`, the promocode name will be `test`

![image](https://user-images.githubusercontent.com/15820577/41859131-642e3f6a-7893-11e8-8708-87e71ae0c8a4.png)

Suggestions and improvements welcome.