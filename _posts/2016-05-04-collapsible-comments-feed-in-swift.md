---
layout: inner
title: 'How to create a collapsible comments feed in Swift 2'
date: 2016-05-04 19:30:34
tags: [tutorial, swift, ios, uitableview, dynamic height]
categories: ios
featured_image: 'https://flyingmoose.co/assets/collapsible-comments-feed-featured.png'
lead_text: 'Learn how to create a collapsible comments feed using UITableView that includes indentation and dynamic table view cell height.'
---

One of the most frustrating challenges I encountered while developing the [Swipe for Reddit](https://flyingmoose.co/swipe) app was implementing the comments feed functionality. My main requirements were that the comments feed:

* Allowed UITableViewCells to be dynamically sized
* Did not lag while scrolling
* Could be collapsed/expanded without relying on "didSelectRowAtIndexPath" (needed that for other functionality within the app)

<br>
While there are [several Objective-C based libraries](https://stackoverflow.com/a/32220026) that exist, I found most were either [abandoned](https://github.com/adamhoracek/KOTree) or [too complex](https://github.com/Augustyniak/RATreeView) for a simple comments tree. After spending several days trying to hack together workarounds for the various issues I encountered, I decided to scrap the 3rd-party libraries all together and just build out my own solution.

### Creating the UITableViewCell

The cell below shows a basic version of what I used in the Swipe app. The important thing here is creating an IBOutlet for the expandRepliesButton's leading constraint, refered to as **expandRepliesButtonLeadingConstraint** (which is just the leading space between expandRepliesButton and it's superview).

{::options parse_block_html="true" /}

<img class="img-responsive" src="{{ site.data.global.url }}/assets/cell-nib.png"/>

Both the username UILabel and the comment UITextView are positioned relative to the **expandRepliesButton**.

### Enabling dynamic height for UITableViewCells

Because the length of a comment varies, and because I wanted to avoid having a scrolling UITextView within each cell, it was necessary for each cell to calculate it's own height. This is actually pretty easy and just requires you to add two bits of code:

*In your UITableViewDelegate methods:*

{% highlight swift %}
func tableView(tableView: UITableView, estimatedHeightForRowAtIndexPath indexPath: NSIndexPath) -> CGFloat {
        return UITableViewAutomaticDimension
    }
{% endhighlight %}

<br>
*Wherever you setup your comments UITableView:*
{% highlight swift %}
tableView.registerNib(UINib(nibName: "LinkCommentCell", bundle: nil), forCellReuseIdentifier: "LinkCommentCell")
tableView.delegate = self
...
// Set the following two properties for your tableView
tableView.estimatedRowHeight = 100.0
tableView.rowHeight = UITableViewAutomaticDimension

{% endhighlight %}


In order for UITableViewCells to be dynamically sized, you must set the **rowHeight** property of your tableView to **UITableViewAutomaticDimension**. Additionally, to minimize lag while scrolling set the **estimatedRowHeight** property to something reasonable since this is what the tableView will use as a baseline when first rendering each cell.

*That's it!*

### Determining the comment's indentation

The final thing you need to include in your comment cell is a way for each cell to determine how indented it should be. I've added a **level** property to my Comment model that represents how much indentation should be applied for each comment. *(ex. level 0 == first comment; level 1 == a comment replying to that comment, etc.)*

In your layout method within the UITableViewCell you will need to use the level property of the cell's comment to calculate the indentation. (I use a [property observer](https://www.hackingwithswift.com/read/8/5/property-observers-didset) for the cell's **comment** property)

{% highlight swift %}
var comment:Comment? {
	didSet {
		populateElements()
	}
}

...

func populateElements() {
	usernameLabel.text = comment!.author
	...
	let indentLevel = CGFloat(comment!.level)
	let indentWidth = 15.0
	expandRepliesButtonLeadingConstraint.constant = CGFloat(indentLevel * indentWidth)

	commentTextView.text = comment!.body
}
{% endhighlight %}

The important thing to note here is that I'm setting the **expandRepliesButtonLeadingConstraint** prior to setting the contents of the **commentTextView**. This ensures that the cell's bounds have been indented properly prior to laying out the **commentTextView**.

Almost finished, just need to add the ability to insert/remove rows from the comments tableView as comment replies are shown/hidden.

### Expanding/collapsing replies

And last but not least, now that you've setup your UITableViewCell for the comments, you need a way for each cell to communicate with the parent tableView that it should add/remove rows when you click on the **expandRepliesButton**. Time for a protocol!

{% highlight swift %}
//Add the ':class' to the protocol declaration to allow the delegate property in the CommentCell to be a weak property
protocol CommentCellDelegate:class {
	func showCommentsReplies(parentComment:Comment, replies:[Comment])
	func hideCommentsReplies(parentComment:Comment, replies:[Comment])
}

...

class CommentCell {
	weak var delegate:CommentCellDelegate?
	...

	@IBAction func expandRepliesButtonTapped(sender: UIButton) {
        if comment.expanded {
        	//the replies property of my Comment model is an array of Commment objects
            delegate?.hideCommentsReplies(comment!, replies:comment!.replies)
            comment.expanded = false
        } else {
            delegate?.showCommentsReplies(comment!, replies:comment!.replies)
            comment.expanded = true
        }
    }
}

{% endhighlight %}
<br>
Make sure your UITableViewDataSource conforms to the **CommentCellDelegate** and add the following in your dataSource class <br/>
*(Note: allComments is the dataSource for the commentsTableView)* :

{% highlight swift %}
extension YourViewController:CommentCellDelegate {
	func showCommentsReplies(parentComment:Comment, replies:[Comment]) {
		//1
		UIView.setAnimationsEnabled(false)

		//2
		tableView.beginUpdates()

		let parentCommentIndex = allComments.indexOf(parentComment)

		for (index, childComment) in replies.enumerate() {
			//3
			let childCommentIndex = parentCommentIndex + (index + 1)

			allComments.insert(comment, atIndex: childCommentIndex)
			let childCommentIndexPath = NSIndexPath(forRow: childCommentIndex, inSection: 0)
			tableView.insertRowsAtIndexPaths([childCommentIndexPath], withRowAnimation: .None)
		}
		//4
		tableView.endUpdates()
		UIView.setAnimationsEnabled(true)
	}
	func hideCommentsReplies(parentComment:Comment, replies:[Comment]) {
		UIView.setAnimationsEnabled(false)
		tableView.beginUpdates()

		let parentCommentIndex = allComments.indexOf(parentComment)

		for (index, childComment) in replies.enumerate() {
			let childCommentIndex = parentCommentIndex + (index + 1)

			allComments.removeAtIndex(childCommentIndex)
			let childCommentIndexPath = NSIndexPath(forRow: childCommentIndex, inSection: 0)
			tableView.deleteRowsAtIndexPaths([childCommentIndexPath], withRowAnimation: .None)
		}
		tableView.endUpdates()
		UIView.setAnimationsEnabled(true)
	}
}
{% endhighlight %}
<br>

1. Set the **setAnimationsEnabled** property of UIView to false before making changes to the comments tableView (otherwise you get [strange visual glitches](https://stackoverflow.com/a/9310886/5329353))
2. Call **tableView.beginUpdates()** to start the update cycle
3. Determine the index of the new/existing child comment and make necessary changes insertions/deletions to the tableView in order (first dataSource, then tableView)
4. Call **tableView.endUpdates()** and enable animations again by calling **UIView.setAnimationsEnabled(true)**

<br>
**And that's all there is!**

Let me know if you have any questions/suggestions either by tweeting me [@FlyingMooseDev](https://twitter.com/FlyingMooseDev) or sending me an email [cam@flyingmoose.co](mailto:cam@flyingmoose.co)
