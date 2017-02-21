---
layout: page
title: About
permalink: /about/
---

```scala
package com.rick.employment


import akka.actor.ActorSystem
import akka.http.scaladsl.model.DateTime
import akka.persistence.cassandra.query.scaladsl.CassandraReadJournal
import akka.persistence.query.PersistenceQuery
import de.heikoseeberger.gabbler.user.Employee.{AddEducation, AddEmployment, GetBasicInfo, SetBasicInfo}

/**
  * Created by rick on 21/2/2017.
  */
object EmployeeApp {


  def main(args: Array[String]): Unit = {
    implicit val system = ActorSystem("employee-system")

    val rick = system.actorOf(Employee.props("RickYoung", PersistenceQuery(system)
      .readJournalFor[CassandraReadJournal](
      CassandraReadJournal.Identifier
    )))

    val setBasicInfo = SetBasicInfo("杨志冲", //名字,曾用名:杨志聪
      DateTime(1992, 4, 2), //生日
      Employee.MALE, //性别
      "中华人民共和国", //国籍
      "广东省广州市花都区花山镇", //地址
      "13922304745", //电话号码
      "contou.y@gmail.com") //邮箱地址

    rick ! setBasicInfo

    /**
      * AddEducation(学校名,专业名,(入学时间,毕业时间))
      */
    val addMiddleSchool = AddEducation("邝维煜纪念中学", "文科",
      (DateTime(2009, 9, 1), DateTime(2011, 7, 6)))
    val addBachelor = AddEducation("南昌理工学院", "工商管理专业",
      (DateTime(2011, 9, 1), DateTime(2015, 7, 6)))

    rick ! addMiddleSchool
    rick ! addBachelor

    val addFirstEmployment = AddEmployment("广州市红甘果信息科技有限公司",
      "安卓开发工程师", Set.empty)
    val addSecondEmployment = AddEmployment("心晴信息科技有限公司",
      "安卓开发工程师", Set.empty)

    rick ! addFirstEmployment
    rick ! addSecondEmployment
  }

}

```

```scala
package com.rick.employment


import akka.actor.{ActorLogging, Props}
import akka.http.scaladsl.model.DateTime
import akka.persistence.PersistentActor
import akka.persistence.query.scaladsl.EventsByPersistenceIdQuery

/**
  * Created by rick on 11/1/2017.
  */
object Employee {
  final val FEMALE: Gender = 0
  final val MALE: Gender = 1

  type Gender = Int
  type SkillSet = Set[String]
  type TimePeriod = (DateTime, DateTime)

  /**
    *
    * @param name      名字
    * @param birthDate 生日
    * @param gender    性别
    * @param country   国籍
    * @param address   地址
    * @param phone     电话好号码
    * @param email     邮箱地址
    */
  case class BasicInfo(name: String,
                       birthDate: DateTime,
                       gender: Gender,
                       country: String,
                       address: String,
                       phone: String,
                       email: String)

  case class SetBasicInfo(name: String,
                          birthDate: DateTime,
                          gender: Gender,
                          country: String,
                          address: String,
                          phone: String,
                          email: String)

  case object GetBasicInfo

  /**
    *
    * @param companyName    公司名字
    * @param positionName   职称
    * @param responsibility 主要任务
    */
  case class Employment(companyName: String,
                        positionName: String,
                        responsibility: Set[String])

  case class AddEmployment(companyName: String,
                           positionName: String,
                           responsibility: Set[String])

  case class DuplicateEmployment(companyName: String)

  case class Employments(employments: Set[Employment])


  case object GetEmployments

  /**
    *
    * @param schoolName 学校名
    * @param major      专业
    * @param period     起始时间-结束时间
    */
  case class Education(schoolName: String,
                       major: String,
                       period: TimePeriod)

  case class AddEducation(schoolName: String,
                          major: String,
                          period: TimePeriod)

  case class DuplicateEducation(schoolName: String)


  case class Educations(educations: Set[Education])

  case object GetEducations


  /**
    * persistent events
    */
  sealed trait EmployeeEvent

  case class BasicInfoSet(basicInfo: BasicInfo) extends EmployeeEvent

  case class EducationAdded(education: Education) extends EmployeeEvent

  case class EmploymentAdded(employment: Employment) extends EmployeeEvent


  def props(name: String, readJournal: EventsByPersistenceIdQuery): Props =
    Props(new Employee(name, readJournal))
}

class Employee(name: String, readJournal: EventsByPersistenceIdQuery)
  extends PersistentActor
    with ActorLogging {

  import Employee._


  // mutable states
  private var basicInfo: BasicInfo = BasicInfo
  private var educations = Map.empty[String, Education]
  private var employments = Map.empty[String, Employment]


  override def receiveRecover: Receive = {
    case EducationAdded(e) => educations += e.schoolName -> e
    case EmploymentAdded(e) => employments += e.companyName -> e
    case BasicInfoSet(info) => basicInfo = info
  }


  override def receiveCommand: Receive = {
    case SetBasicInfo(n, b, g, c, a, p, e) => handleSetBasicInfo(n, b, g, c, a, p, e)
    case AddEducation(schoolName, major, period) => handleAddEducation(schoolName, major, period)
    case AddEmployment(n, p, r) => handleAddEmployment(n, p, r)
    case GetBasicInfo => sender() ! basicInfo
    case GetEmployments => sender() ! Employments(employments.valuesIterator.to[Set])
    case GetEducations => sender() ! Educations(educations.valuesIterator.to[Set])
  }

  /**
    * add the  education info to the repo and persist it
    * if the education info was not already existed
    *
    * @param schoolName
    * @param major
    * @param period
    */
  def handleAddEducation(schoolName: String,
                         major: String,
                         period: TimePeriod) = {
    if (educations.contains(schoolName)) {
      sender() ! DuplicateEducation(schoolName)
    } else {
      persist(EducationAdded(Education(schoolName, major, period))) {
        educationAdded =>
          receiveRecover(educationAdded)
          sender() ! educationAdded
      }
    }
  }

  /**
    * add the  employment info to the repo and persist it
    * if the employment was not already existed
    *
    * @param companyName
    * @param positionName
    * @param responsibility
    */
  def handleAddEmployment(companyName: String,
                          positionName: String,
                          responsibility: Set[String]) = {
    if (employments.contains(companyName)) {
      sender() ! DuplicateEmployment(companyName)
    } else {
      persist(EmploymentAdded(Employment(companyName, positionName, responsibility))) {
        employmentAdded =>
          receiveRecover(employmentAdded)
          sender() ! employmentAdded
      }
    }
  }

  /**
    * check if the basicInfo was valid
    * if so replace the old basicInfo with the new one
    *
    * @param name
    * @param birthDate
    * @param gender
    * @param country
    * @param address
    * @param phone
    * @param email
    */
  def handleSetBasicInfo(name: String,
                         birthDate: DateTime,
                         gender: Gender,
                         country: String,
                         address: String,
                         phone: String,
                         email: String) = {
    //TODO check if the info was valid
    if (true) {
      val info = BasicInfo(name, birthDate,
        gender, country, address, phone, email)
      
      persist(BasicInfoSet(info)) {
        basicInfoSet =>
          receiveRecover(basicInfoSet)
          sender() ! basicInfoSet
      }
    }
  }

  override def persistenceId: String = "Employee_" + name
}

```