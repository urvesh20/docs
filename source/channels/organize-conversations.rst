Organize conversations using Collapsed Reply Threads
====================================================

|all-plans| |cloud| |self-hosted|

.. |all-plans| image:: ../images/all-plans-badge.png
  :scale: 30
  :target: https://mattermost.com/pricing
  :alt: Available in Mattermost Free and Starter subscription plans.

.. |cloud| image:: ../images/cloud-badge.png
  :scale: 30
  :target: https://mattermost.com/sign-up
  :alt: Available for Mattermost Cloud deployments.

.. |self-hosted| image:: ../images/self-hosted-badge.png
  :scale: 30
  :target: https://mattermost.com/deploy
  :alt: Available for Mattermost Self-Hosted deployments.

.. |more-actions-icon| image:: ../images/dots-horizontal_F01D8.svg
  :height: 24px
  :width: 24px
  :alt: Access additional message actions using the More Actions icon.

Threads are a key part of the messaging experience in Mattermost Channels. They're used to organize conversations and enable users to discuss topics without adding noise to channels or direct messages.

Collapsed Reply Threads offers an enhanced experience for users communicating in threads and replying to messages that includes a unified threads inbox to read all conversations in one view. Threads improve the ability to process channel content, find, follow, and resume conversations more easily, and keep threaded conversations focused.

.. image:: ../images/collapsed-reply-threads.gif
  :alt: Organize conversations using Collapsed Reply Threads.

Start or reply to threads
-------------------------

Replies are collapsed under the first message of a thread. To reply to a thread, select the **Reply** icon, or select the reply count if a thread already exists.

.. tip:: 
    
    - Select anywhere on a message in a channel in the center pane to view it, or reply to it on the right-hand side.
    - In channels, a dot next to the thread participants indicates there are unread replies. You'll only see unreads for threads you're following.

.. image:: ../images/crt-new-unread-threads.jpg
   :alt: A dot on threads in a channel indicates unread replies.

Follow threads and messages
---------------------------

You can follow particular messages and threads so that any reply activity triggers `notifications <https://docs.mattermost.com/channels/channels-settings.html#notifications>`__. Follow or unfollow any thread, at any time, by toggling the thread’s **Follow/Following** indicator, or from the **More Actions** |more-actions-icon| icon.

.. image:: ../images/crt-following-thread.png
   :alt: Follow threads to stay updated on replies to messages.

You'll automatically follow every thread you participate or are mentioned in. If you’re no longer interested in a or message thread, you can unfollow it to stop receiving notifications. Viewing a thread without responding to it doesn’t automatically follow that thread.

.. image:: ../images/crt-following-thread.jpg
   :alt: Follow, unfollow, and mark threads as unread from the More Actions icon.
   
.. tip::
  - Follow messages with no replies from the **More Actions** |more-actions-icon| icon to be notified if someone replies to the message later based on your notification preferences.
  - Use keyboard arrow keys to navigate threads in the **Threads** view.

View all threads
----------------

Select **Threads** at the top of the channel sidebar to see all your followed threads on the currently selected team. Threads with the most recent replies display at the top of the list. 

Select **Unreads** to filter your followed threads by only those with unread replies.

.. image:: ../images/crt-thread-view.jpg
  :alt: Select Threads in the channel sidebar to see all thread updates in your Threads View.

Enable Collapsed Reply Threads
------------------------------

See our `configuration settings <https://docs.mattermost.com/configure/configuration-settings.html#collapsed-reply-threads>`__ documentation for details on configuring the default availability of collapsed reply threads. From Mattermost v7.0, Collapsed Reply Threads are enabled by default for all new Mattermost deployments.

Depending on how your System Admin has configured **Collapsed Reply Threads** for your workspace, it may already be enabled for you, or you may be able to `enable this feature for your account <https://docs.mattermost.com/channels/channels-settings.html#collapsed-reply-threads>`__ by going to **Settings > Display > Collapsed Reply Threads**. 

Tutorial video
---------------

.. raw:: html

  <script src="https://fast.wistia.com/embed/medias/5gjgi10rr0.jsonp" async></script><script src="https://fast.wistia.com/assets/external/E-v1.js" async></script><div class="wistia_responsive_padding" style="padding:56.25% 0 0 0;position:relative;"><div class="wistia_responsive_wrapper" style="height:100%;left:0;position:absolute;top:0;width:100%;"><div class="wistia_embed wistia_async_5gjgi10rr0 videoFoam=true" style="height:100%;position:relative;width:100%"><div class="wistia_swatch" style="height:100%;left:0;opacity:0;overflow:hidden;position:absolute;top:0;transition:opacity 200ms;width:100%;"><img src="https://fast.wistia.com/embed/medias/5gjgi10rr0/swatch" style="filter:blur(5px);height:100%;object-fit:contain;width:100%;" alt="" aria-hidden="true" onload="this.parentNode.style.opacity=1;" /></div></div></div></div>
  <br>


Known issues
------------

Collapsed Reply Threads was released as generally available in Mattermost v7.0, including significant server performance improvements and more flexible configuration options for System Admins to enable the feature by default. We highly recommended `upgrading to Mattermost v7.0 <https://docs.mattermost.com/upgrade/upgrading-mattermost-server.html>`__ to take advantage of these configuration and performance enhancements.

Customers running Collapsed Reply Threads in beta on Mattermost v6.7 or earlier will likely experience bugs and unoptimized server performance. Learn more about the `beta release performance considerations <https://support.mattermost.com/hc/en-us/articles/4413183568276>`__ applicable only to Mattermost v5.37 through v6.7 deployments.

In particular, please be aware of these important known issues and risks:

  - When enabling Collapsed Reply Threads for the first time, you may see channels or threads you’ve seen before appear as unread. To resolve this, navigate to any unread channel, or select the **Mark all as read** button in the **Threads** view to mark all of your threads as read.
  - You may experience lag in your desktop or web client if you're following many threads or opening threads with many replies.
  - Server performance for instances with many users and posts may be impacted. 
  - You should expect to see more unread channels and notifications on your mobile device than you'll see on your desktop instance of Mattermost. This is expected when running a mobile app release older than v1.46.
  - For a comprehensive list of known issues, and to see our work queue in priority order, check out our `Kanban board <https://mattermost.atlassian.net/secure/RapidBoard.jspa?rapidView=91&quickFilter=499>`__.