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