* functor
** Decoder[A], Json => Res[A], f: A => B
** Encoder[A], A => Json, f: B => A

(starting `sbt console`)

s.11
implicit val encodeUserId: Encoder[UserId] = Encoder[String].contramap(_.value)
implicit val decodeUserId: Decoder[UserId] = io.circe.Decoder[String].map(UserId(_))

(after) s.12, 13
jawn.decode(bad)(decodeUserMonadic)
>res1.left.get
<res2: io.circe.Error = DecodingFailure(String, List(DownField(first)))

>res1.left.get.show
<res3: String = DecodingFailure at .first: String

s.14
idea: finish first two decodings, before run third decoding and after that map Strings of User class(first, last, age)
 -> so (for compehension) fail first

s.15
idea: val decodeUserApplicative: Decoder[User] =  (decodeFirst, decodeLast, decodeAge).mapN(User(_, _, _))
-> map it directly without having a data depdency like first has been finished before last would be mapped


s.18
>decodeUserNoExtras.decodeJson(json"""{"first":"A", "last": "B", "age": 25, "extra": []}""")
<res5: io.circe.Decoder.Result[User] = Left(DecodingFailure(Leftover keys: extra, List(DeleteGoParent, DownField(age), DeleteGoParent, DownField(last), DeleteGoParent, DownField(first))))

>decodeUserNoExtras.decodeJson(json"""{"first":"A", "last": "B", "age": 25, "extra": [], "another": 10}""")
<res6: io.circe.Decoder.Result[User] = Left(DecodingFailure(Leftover keys: extra, another, List(DeleteGoParent, DownField(age), DeleteGoParent, DownField(last), DeleteGoParent, DownField(first))))


s.20 Generic derivation
get a User obj decoder by derivation from decoder of Int and String?

>User("Foo", "McBar", 25).asJson
-> every time the compiler creates a new UserDecoder by each call 
-> by using the following input, it uses the created UserDecoder:
>implicit val encodeUser: Encoder[User] = io.circe.generic.semiauto.deriveEncoder

s.21
usage: create a user with using given information the user inputs, but add one thing like a user id or so
-> don't use a case class without Option[Id] type!
-> more information: https://meta.plasm.us/posts/2015/06/21/deriving-incomplete-type-class-instances/

s.22
configure how the json key names looks like -> so you can create the names in your case class with camel case
 and use json key names in snake case style   


s.26/27
need the typelevel scala compiler https://github.com/typelevel/scala or use the shapeless import as on the slide...


s.27
-> define more granular types
User("A", "B", -1)
<console>:48: error: Predicate failed: (-1 > 0).
       User("A", "B", -1)


... more s.27... wtf:
case class User(name: String, age: Refined[Int, boolean.And[Greater[10], Less[100]]])
defined class User

scala> User("Foo", 100)
<console>:48: error: Right predicate of ((100 > 10) && (100 < 100)) failed: Predicate failed: (100 < 100).
       User("Foo", 100)

--

>type C = boolean.And[Greater[10], Less[100]]
>refineV[C](x)
<Either[String,eu.timepit.refined.api.Refined[Int,C]] = Right(11)
>refineV[C](x).map(User("ABC", _))
<scala.util.Either[String,User] = Right(User(ABC,11))

more examples: https://github.com/fthomas/refined
