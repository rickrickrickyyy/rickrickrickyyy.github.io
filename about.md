---
layout: page
title: About
permalink: /about/
---

```scala
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