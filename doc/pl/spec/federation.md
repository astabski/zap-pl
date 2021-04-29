Federacja
=========

Implementacja ActivityPub w tym projekcie stara się być zgodna z podstawową specyfikacją tam, gdzie to możliwe, oferując jednocześnie szereg usług i funkcji, które normalnie nie są dostarczane przez projekty ActivityPub.

Bezpośrednie wiadomości
-----------------------

Wiadomości bezpośrednie (Direct Messages - DM) rózróżniane są od innych wiadomości prywatnych za pomocą flagi `zot:directMessage` (boolean). Jest to zgodne z tą samą funkcją zapewnianą przez inne projekty w innych przestrzeniach nazw i nie jest poprzedzone prefiksem w ramach działań, aby mogły one potencjalnie zostać zagregowane.

Wydarzenia
----------

Wydarzenia i odpowiedzi RSVP sa obsługiwane zgodnie ze słownictwem AS z takim wyjątkiem, że Create/Event nie jest przesyłane. Invite/Event jest podstawową metodą odostępniania wydarzeń. W celu zapewnienia zgodności ze starszymi aplikacjami, odpowiedzi RSVP metod Accept/Event, Reject/Event, TentativeAccept/Event i TentativeReject/Event sa akceptowane jako prawidłowe aktywności RSVP. Domyślnie wysyłamy Accept/{Invite/Event} (i inne odpowiedzi RSVP) według słownictwa AS. Wydarzenia bez strefy czasowej (np. "wydarzenia całodniowe" lub święta) sa wysyłane bez 'Z' w znaczniku czasowym zgodnie z RFC3339 Section 4.3. Wszystkie znaczniki czasowe wydarzeń są podawana w czasie UTC , a zdarzenia dostosowane do strefy czasowej są transmitowane przy użyciu czasu Zulu, 'yyyy-mm-ddThh:mm:ssZ'. Opisy wydarzeń są wysyłane przy użyciu pola 'summary' i akceptują użycie pól 'summary', 'title' i 'content' w kolejności istnienia. Jeśli zawierją kod HTML, to są przekształcane wewnętrznie do zwykłego tekstu. Jeśli pole 'location' zawiera współrzędne, podczas renderowania jest generowana mapa. 


Tożamość nomadyczna
-------------------

Nomadic identity describes a mechanism where the data stored by your social network can be mirrored to multiple servers in near realtime and any of these servers can be used at any time should your "primary" server be unavailable (temporarily or permanently). The mechanisms for creating nomadic accounts and maintaining mirrors are currently not supported by ActivityPub but are available via the Zot6 protocol. However, the instance information is provided to ActivityPub services which wish to support it in a more seamless manner. Otherwise DMs (for example) may need to be sent to all actor instances to ensure the nomadic actor receives them. There are many other examples. 

The actor record for nomadic accounts contains a 'copiedTo' property which is otherwise identical to Mastodon's 'movedTo' property. ActivityPub services wishing to support compatibility with this mechanism should ensure that each nomadic instance is "folded" into a single fediverse identity so that messages from the same actor across different instances are recognised as being the same author. Additionally, any posts sent to a nomadic account should be sent to all instances and private data should be made accessible to all actor instances. This includes historical data and posts as new instances can be created at any time.  

Grupy
-----

Groups may be public or private. The initial thread starting post to a group is sent using a DM to the group and should be the only DM recipient. This helps preserve the sanctity of private groups and is a posting method available to most ActivityPub software, as opposed to bang tags (!groupname) which lack widespread support and normal @mentions which can create privacy issues and their associated drama.  It will be converted to an embedded post authored by the group Actor (and attributed to the original Actor) and resent to all members. Followups and replies to group posts use normal federation methods. The actor type is 'Group' and can be followed using Follow/Group *or* Join/Group, and unfollowed by Undo/Follow *or* Leave/Group.

Update: as of 2021-04-08 @mentions are now permitted for posting to public and moderated groups but are not permitted for posting to  restricted or private groups. The group owner can over-ride this behaviour as desired based on the group's security and privacy expectations. DMs (and wall-to-wall posts) are still the recommended methods for posting to groups because they can be used for any groups without needing to remember which are public and which are private; and which may have allowed or disallowed posting via mentions. 

Komentarze
----------

This project provides permission control and moderation of comments. By default comments are only accepted from existing connections. This can be changed by the individual. Other sites MAY use zot:commentPolicy (string) as a guide if they do not wish to provide comment abilities where it is known in advance they will be rejected. A Reject/Note activity will be sent if the comment is not permitted. There is currently no response for moderated content, but will likely also be represented by Reject/Note.

'commentPolicy' can be any of

'authenticated' - matches the typical ActivityPub permissions

'contacts' - matches approved followers

'any connections' - matches followers regardless of approval

'site: foobar.com' - matches any actor or clone instance from 'foobar.com'

'network: red' - matches any actor from the 'red' network

'public' - matches anybody at all, may require moderation if the network isn't known

'self' - matches the activity author only

'until=2001-01-01T00:00Z' - comments are closed after the date given. This can be supplied on its own or appended to any other commentPolicy string by preceding with a space; for example 'contacts until=2001-01-01T00:00Z'.


Media prywatne
--------------

Private media MAY be accessed using OCAP or OpenWebAuth.


System uprawnień
----------------

The Zot permission system has years of historical use and is different than and the reverse of the typical ActivityPub project. We consider 'Follow' to be an anti-pattern which encourages pseudo anonymous stalking. A Follow activity by an actor on this project typically means the actor on this project will send activities to the recipient. It may also confer other permissions. Accept/Follow usually provides permission to receive content from the referenced actor, depending on their privacy settings. 


Model dostawy
-------------

This project uses the relay system pioneered by projects such as Friendica, Diaspora, and Hubzilla which attempts to keep entire conversations intact and keeps the conversation initiator in control of the privacy distribution. This is not guaranteed under ActivityPub where conversation members can cc: others who were not in the initial privacy group. We encourage projects to not allow additional recipients or not include their own followers in followups. Followups SHOULD have one recipient - the conversation owner or originator, and are relayed by the owner to the other conversation members. This normally requires the use of LD-Signatures but may also be accessible through authenticated fetch of the activity using HTTP signatures. 

Treść
-----

Content may be rich multi-media and renders nicely as HTML. Multicode is used and stored internally. Multicode is html, markdown, and bbcode. The multicode interpreter allows you to switch between these or combine them at will. Content is not obviously length-limited and authors MAY use up to the storage maximum of 24MB. In practice markup conversion limits the effective length to around 200KB and the default "maximum length of imported content" from other sites is 200KB. This can be changed on a per-site basis but this is rare. A Note may contain one or more images, other media, and links. HTML purification primarily removes javascript and iframes and allows most legitimate tags and CSS styles through. Inline images are also added as attachments for the benefit of Mastodon (which strips most HTML), but remain in the HTML source. When importing content from other sites, if the content contains an image attachment, the content is scanned to see if a link (a) or (img) tag containing that image is already present in the HTML. If not, an image tag is added inline to the end of the incoming content. Multiple images are supported using this mechanism.

Mastodon 'summary' does not invoke any special handling so 'summary' can be used for its intended purpose as a content summary. Mastodon 'sensitive' is honoured and results in invoking whatever mechanisms the user has selected to deal with this type of content. By default images are obscured and are 'click to view'. Sensitive text is not treated specially, but may be obscured using the NSFW plugin or filtered per connection based on string match, tags, patterns, languages, or other criteria.

Edycje
------

Edited posts and comments are sent with Update/Note and an 'updated' timestamp along with the original 'published' timestamp. 

Ogłoszenia
----------

Announce and relay activities use two mechanisms. As wll as the Announce activity, a new message is generated with an embedded rendering of the shared content as the message content. This message may (should) contain additional commentary in order to comply with the Fair Use provisions of copyright law. The other reason is our use of comment permissions. Comments to Announce activities are sent to the author (who typically accepts comments only from connections). Comment to embedded forwards are sent to the sender. This difference in behaviour allows groups to work correctly in the presence of comment permissions.

Discussion (2021-04-17): In the email world this type of conflict is resolved by the use of the reply-to header (e.g. in this case reply to the group rather than to the author) as well as the concept of a 'sender' which is different than 'from' (the author). We will soon be modelling the first one in ActivityPub with the use of 'replyTo'. If you see 'replyTo' in an activity it indicates that replies SHOULD go to that address rather than the author's inbox. We will implement this first and come up with a proposal for 'sender' if this gets any traction. If enough projects support these constructs we can eliminate the multiple relay mechanisms and in the process make ActivityPub much more versatile when it comes to organisational and group communications. Our primary use case for 'sender' is to provide an ActivityPub origin to a message that was imported from another system entirely (such as Diaspora or from RSS source). In this case we would set 'attributedTo' to the remote identity that authored the content, and 'sender' to the person that posted it in ActivityPub. 

Niestandardowe emotikony Mastodona
----------------------------------

Mastodon Custom Emojis are only supported for post content. Display names and message titles are considered text only fields and embedded images (the mechanism behind custom emojis) are not supported in these locations.


Wzmianki i wzmianki prywatne
----------------------------

By default the mention format is '@Display Name', but other options are available, such as '@username' and both '@Display Name (username)'. Mentions may also contain an exclamation character, for example '@!Display Name'. This indicates a Direct or private message to 'Display Name' and post addressing/privacy are altered accordingly. All incoming and outgoing content to your stream is re-written to display mentions in your chosen style. This mechanism may need to be adopted by other projects to provide consistency as there are strong feelings on both sides of the debate about which form should be prevalent in the fediverse.  


Powiadomienia o komentarzach (Mastodon)
---------------------------------------

Our projects send comment notifications if somebody replies to a post you either created or have previously interacted with in some way. They also are able to send a "mention" notification if you were mentioned in the post. This differs from Mastodon which does not appear to support comment notifications at all and only provides mention notifications. For this reason, Mastodon users don't typically get notified unless the author they are replying to is mentioned in the post. We provide this mention in the 'tag' field of the generated Activity, but normally don't include it in the message body, as we don't actually require mentions that were created for the sole purpose of triggering a comment notification.

Kompletacja rozmów
------------------

(2021-04-17) Łatwo jest pobrać brakujące fragmenty rozmowy przechodzącej "w górę", ale nie ma uzgodnionej metody pobierania całej rozmowy z punktu widzenia aktora źródłowego, a pobieranie z góry zapewnia tylko jedną gałąź konwersacji, a nie całe drzewo. Podajemy "kontekst" jako adres URL do zbioru zawierającego całą rozmowę (wszystkie znane gałęzie) widzianą przez jej twórcę. Wymaga to specjalnego traktowania i jest bardzo podobne do ostatus: rozmowy w tym sensie, że jeśli obecny jest kontekst, to trzeba ją powtórzyć w rozmowie potomków. Nadal wspieramy ostatus: rozmowę, ale użycie jest przestarzałe. Nie używamy "odpowiedzi" do osiągnięcia tych samych celów, ponieważ "odpowiedzi" odnoszą się tylko do bezpośrednich potomków w dowolnym punkcie drzewa konwersacji.
