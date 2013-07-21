Following are changes I made to the Spark source code.

# where is akka used.


$ grep -r --include=*.scala "akka.actor" *  > log.txt
$ grep -r --include=*.scala "akka" *  > log2.txt


# Changes to the Library Implementation

## play.core.j.OrderedExecutionContext.scala

line 3:
import akka.actor.{ Actor, ActorSystem, Props }  ->
import takka.actor.{ TypedActor, ActorSystem, Props }

line 17:
  private val actors = Array.fill(size)(actorSystem.actorOf(Props[OrderedExecutionContext.RunActor])) ->
  private val actors = Array.fill(size)(actorSystem.actorOf(Props[Runnable, OrderedExecutionContext.RunActor]))

line 33:
  class RunActor extends Actor { ->
  class RunActor extends TypedActor[Runnable] {




## play.core.j.OrderedExecutionContextSpec.scala

line 2: 
import akka.actor.ActorSystem  ->
import takka.actor.ActorSystem

line 34:
    def receive = {  ->
    def typedReceive = {

## play.core.Invoker.scala

line 3:
import akka.actor._ ->
import takka.actor._

## play.libs.Akka.java
### add play.libs.Akka.TAkka.java by changing
line 3:
import akka.actor.ActorSystem; ->
import takka.actor.ActorSystem; 

line 16:
public class Akka {  ->
public class TAkka {



line 32:
return play.api.libs.concurrent.Akka.system(Play.current());  ->
return play.api.libs.concurrent.TAkka.system(Play.current());
## play.api.libs.concurrent.Akka.scala
### add play.api.libs.concurrent.TAkka.scala by changing
line 6:
import akka.actor.ActorSystem ->
import takka.actor.ActorSystem

line 13:
object Akka { ->
object TAkka {

line 24:
app.plugin[AkkaPlugin].map(_.applicationSystem).getOrElse { ->
app.plugin[TAkkaPlugin].map(_.applicationSystem).getOrElse {

line 49
class AkkaPlugin(app: Application) extends Plugin { ->
class TAkkaPlugin(app: Application) extends Plugin {

## play.core.server.Server.scala

line 13 and 14:
import takka.actor._
import akka.actor.Actor._  ->

import takka.actor._
import takka.actor.Actor._


# sbt test passed

# ./runtest passed

## Add test for using TAKKA actors

### /documentation/manual/scalaGuide/main/akka/code/ScalaTAkka.scala

modified from /documentation/manual/scalaGuide/main/akka/code/ScalaAkka.scala

line 10:
import akka.actor.{Actor, Props} ->
import takka.actor.{TypedActor, Props}

line 14:
import play.api.libs.concurrent.Akka ->
import play.api.libs.concurrent.TAkka


line 28, 40, 52:
val testActor = Akka.system.actorOf(Props[MyActor], name = "testActor")  ->
val testActor = TAkka.system.actorOf(Props[String, MyActor], "testActor")

line 44:

Akka.system.scheduler.schedule(0.microsecond, 300.microsecond, testActor, "tick")  ->
TAkka.system.scheduler.schedule(0.microsecond, 300.microsecond, testActor, "tick")

line 58:
Akka.system.scheduler.scheduleOnce(1000.microsecond) { ->
TAkka.system.scheduler.scheduleOnce(1000.microsecond) {

line 72 - 76
class MyActor extends Actor {
  def typedReceive = {
    case s: String => sender ! "Hello," + s
  }
}

->

class MyActor extends TypedActor[String] {
  def typedReceive = {
    case s: String => sender ! "Hello," + s
  }
}

