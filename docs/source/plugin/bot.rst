=====================
Interact with the bot
=====================

Once a :term:`Rule` has been triggered, it's time to do whatever the plugin is
supposed to do. Thanks to the ``bot`` parameter, you can make the bot talk:
say something in a channel, reply to someone, send a notice, or join a channel.

.. contents::
   :local:
   :depth: 2


Make it talk
============

The most basic way to make the bot talk is to use its
:meth:`~sopel.bot.SopelWrapper.say` method. The wrapper knows the origin of
the trigger (a channel or a private message), and it will use this origin as
the default destination for your message::

    # will send that to trigger.sender
    bot.say('The bot is now talking!')

If you want to send the message to another destination, you can pass it as the
second argument::

    bot.say('The bot is now talking!', '#private-channel')

Instead of a string, you can use an instance of :class:`sopel.tools.Identifier`.

If you want to reply to a user in a private message, you can use the trigger's
:attr:`~sopel.trigger.Trigger.nick` attribute as destination::

    bot.say('I reply in private message', trigger.nick)

And if you want to send a private message to the bot's owner every time your
rule is triggered, you can use the bot's settings::

    bot.say('Hi owner!', bot.settings.core.owner)

.. note::

    The ``say`` method sends a ``PRIVMSG`` command to the IRC server. To send
    a ``NOTICE`` command instead, you need to use the
    :meth:`~sopel.bot.SopelWrapper.notice` method instead.


Make it reply
=============

Now maybe you want to make sure the user gets notified by the bot's message.
For that, you could use ``trigger.nick`` this way::

    bot.say('%s: ping!' % trigger.nick)

It'll work fine and it's a common usage. So common indeed that Sopel provides a
shortcut for that::

    bot.reply('ping!')

As with the ``say`` method seen above, the :meth:`~sopel.bot.SopelWrapper.reply`
method can send your message to another destination::

    bot.reply('ping!', '#another-channel')

Also, if you want to reply to **someone else**, you can do that too by using
the ``reply_to`` parameter::

    bot.reply('ping!', reply_to=bot.settings.core.owner)

In that example, we send a message on the same channel, with a highlight to the
bot's owner.

.. note::

    By default the ``reply`` method sends its message using a ``PRIVMSG``
    command. You can set ``notice=True`` as argument to make it use a
    ``NOTICE`` command instead::

        bot.reply('ping!', notice=True)


Make it act
===========

Besides talking, the bot can also **act**:

* to :meth:`~sopel.bot.Sopel.join` a channel,
* or to :meth:`~sopel.bot.Sopel.part` from it,
* and even to :meth:`~sopel.bot.Sopel.quit` the server,

Oh, and let's not forget about ``/me does something``, which can be done with
the :meth:`~sopel.bot.SopelWrapper.action` method::

    bot.action('does something')


Do it with style
================

.. Custom role definitions to apply custom style to inline text

.. role:: red
    :class: red

.. role:: boldred
    :class: bold red

.. role:: underline
    :class: underline

.. role:: strike
    :class: strike

.. role:: bolditalic
    :class: bold italic

.. role:: spoiler
    :class: spoiler


When the bot talks, replies, or acts, it can do so with style: colors,
**bold**, *italic*, :underline:`underline`, :strike:`strikethrough`, or
``monospace``. IRC formatting works with control codes, bytes you can use to
tell IRC clients how to display some part of the text.

.. seealso::

    If you want to know more about IRC formatting in general and some of its
    limitations, `the modern IRC documentation`__ may be of interest to you.

    .. __: https://modern.ircdocs.horse/formatting.html

However, dealing with control codes yourself is not the most dev-friendly
approach, hence the :mod:`sopel.formatting` module. It contains various
functions to help you create styled text.

Text styles
-----------

Let's dive into examples, starting with :func:`~sopel.formatting.bold` text::

    from sopel import formatting

    bot.say(formatting.bold('This is some bold text!'))

This will output a line like this:

    <Sopel> **This is some bold text!**

You can use them with Python string formatting::

    emphasis = formatting.bold('important')
    bot.say('And here is the %s part.' % emphasis)

To get that kind of output:

    <Sopel> And here is the **important** part.

And you can use multiple style functions together, for example with the
:func:`~sopel.formatting.italic` function::

    word = formatting.italic('very')
    emphasis = formatting.bold('%s important' % word)
    bot.say('And here is the %s part.' % emphasis)

To get a result that looks like this:

    <Sopel> And here is the :bolditalic:`very` **important** part.

Colored styles
--------------

Colorized text is a bit more complicated, and Sopel tries to provide helpful
functions and constants for that: the :func:`~sopel.formatting.color` function
and the :class:`~sopel.formatting.colors` class.

The ``color`` function takes a line of text and a foreground color. It also
accepts an optional background color that uses the same color codes. The color
codes are listed by the ``colors`` class, and can be used like this::

    bot.say(formatting.color('Red text.', formatting.colors.RED))

The above example should produce this output:

    <Sopel> :red:`Red text.`

You can combine colors and styles, like this::

    big = formatting.color(
        formatting.bold('WARNING'), formatting.colors.RED)
    small = formatting.italic('warning')
    bot.say('[%s] This is a %s.' % (big, small))

So you get a similar result as:

    <Sopel> [:boldred:`WARNING`] This is a *warning*.

If you want to prevent spoilers, you could be tempted to take advantage of
the background color::

    spoiler = formatting.color(
        'He was the killer.',
        formatting.colors.BLACK,
        formatting.colors.BLACK,
    )
    bot.say(spoiler)

And expect this (you need to select the text to read it):

    <Sopel> :spoiler:`He was the killer.`

Note that not all combinations of foreground and background colors are happy
ones, and you should be mindful of using too many unnecessary colors.


Channels & users
================

Knowing how to talk is good for a bot, but you may be wondering what the bot
knows about the channels and their users. For that, you can use the bot's
:attr:`~sopel.bot.Sopel.channels` attribute. For example, to list all channels
the bot is in::

    for name, channel in bot.channels.items():
        # do something with the name and the channel

With the ``trigger`` object, you can also access the channel object directly
(assuming the message comes from a channel, which you should check first)::

    channel = bot.channels[trigger.sender]

The ``channel`` object is an instance of :class:`sopel.tools.target.Channel`,
which provides the following information:

* its :attr:`~sopel.tools.target.Channel.name`
* its :attr:`~sopel.tools.target.Channel.topic`
* its :attr:`~sopel.tools.target.Channel.users`
* and its users' :attr:`~sopel.tools.target.Channel.privileges`

Using ``trigger.nick``, you can get the nick's privileges and profile in a
channel like this::

    user_privileges = channel.privileges[trigger.nick]
    user = channel.users[trigger.nick]

Then, for example, you can check if the user is voiced (mode +v) or not::

    from sopel import plugin

    if user_privileges & plugin.VOICED:
        # user is voiced
    elif user_privileges > plugin.VOICED:
        # not voiced, but higher privileges
        # like plugin.HALFOP or plugin.OP
    else:
        # no privilege
