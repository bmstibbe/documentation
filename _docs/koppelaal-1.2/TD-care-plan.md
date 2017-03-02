---
title: Care Plan Management
category: Technical Designs
order: 1
---

Author: Bart Mehlkop
Datum: 02-02-2017
Versie: 0.1

# Version management

<table>
  <tr>
    <td>Versie</td>
    <td>Datum</td>
    <td>Auteur</td>
    <td>Aanpassingen</td>
  </tr>
  <tr>
    <td>0.1</td>
    <td>02-12-2016</td>
    <td>Bart Mehlkop</td>
    <td>First Draft</td>
  </tr>
  <tr>
    <td>1.0</td>
    <td>07-12-2016</td>
    <td>Bart Mehlkop</td>
    <td>Finalized TD</td>
  </tr>
 </table>

# Table of content

[[TOC]]

# Use case

A CarePlan is typically created by a Practitioner in a practitioner portal. Any applications in the Domain that offer activities check whether or not an activity of theirs is included in the CarePlan. If so, they may retrieve any nescessary data from the CarePlan and store it for use later, when the Patient starts the activity.

At various moments the status of an activity may change, such as when a Patient starts or completes an activity. Whenever a status change happens, the activity provider (intervention, game, etc) sends an UpdateCarePlanActivityStatus message that the CarePlan owner (practitioner portal) uses to update the CarePlan. The CarePlan owner then sends a CreateOrUpdateCarePlan message to inform all applications of the change. Additionally, an activity provider can send a CreateOrUpdateCarePlanActivityResults message to update other applications with the medical results generated by the player's activities.

An UpdateCarePlanActivityStatus message is a subset of the CreateOrUpdateCarePlan message. This message is intended for use by applications that have no need to store the entire CarePlan. With the UpdateCarePlanActivityStatus message they only need to store data relevant for their own activity. Because the status of different activities can be dependent (for example, in the case that activity B is only made available after activity A is finished) the CarePlan owner is responsible for sending CreateOrUpdateCarePlan messages with the full CarePlan status whenever an activity status changes.


# Example message flow

The main source of sensitive data on the Koppeltaal Server are the FHIR messages that are exchanged between applications. These messages may contain all sorts of patient data, including name, BSN and address information. 

There are a number of related sets of data that contain patient information as a side effect of their function. Specificially, the table ‘IntegrationMessageLogs’ logs all API-calls, including those for posting new FHIR messages, and includes the content of the call.

The ‘IntegrationMessageLogs’ table is already being cleaned automatically, with calls that completed successfully being cleaned after 7 days and failed calls after 14 days. These records are useful when debugging any issues that occur on the Koppeltaal Server.

# Requirements


In this example we will assume the following fictitious situation: healthcare organisation HealthOnline will start offering the game 'Patients vs. Pollen' to their patients. This game provides tips and tricks on how to deal with hay fever in the form of a humorous game. The caregivers of HealthOnline will log in to the practitioner portal 'CaregiversOnline', where they send an invitation for them to play the game. Patients can log in to the patient portal 'MyHealthOnline' where they can find a link to play the 'Patients vs. Pollen' game. For this example it is assumed that Patients are already known in the patient portal.



# Example message flow
In this example we will assume the following fictitious situation: healthcare organization HealthOnline will start offering the game 'Patients vs. Pollen' to their patients. This game provides tips and tricks on how to deal with hay fever in the form of a humorous game. The caregivers of HealthOnline will log in to the practitioner portal 'CaregiversOnline', where they send an invitation for them to play the game. Patients can log in to the patient portal 'MyHealthOnline' where they can find a link to play the 'Patients vs. Pollen' game. For this example it is assumed that Patients are already known in the patient portal.

  - Koppeltaal Server configuration
  
The above situation leads to the following Koppeltaal Server configuration:

Domain name: HealthOnline

Applications:
| Name  | Identifier  |  Role |  Subscribed message types |   |
|---|---|---|---|---|
|  Patients vs. Pollen | PVP  | Game  |  CreateOrUpdateCarePlan |   |
| CaregiversOnline  |  CO | Practitioner Portal  |  UpdateCarePlanActivityStatus, CreateOrUpdateCarePlanActivityResult |   |
| MyHealthOnline  | MHO  |Patient Portal   |  CreateOrUpdateCarePlan, CreateOrUpdateCarePlanActivityResult |   |

The Patients vs. Pollen application provides only one ActivityDefinition:


| Name |	ActivityType |	Identifier |	Description	IsActive |	Default performer |
|---|---|---|---|---|
| Patients vs. Pollen| 	Game |	PVPGAME	Learn to fight the pollen armies.|	True|	Patient|

# Message flow
The diagram below gives an overview of the entire sequence of messages exchanged between applications. The exact contents of the messages along with an explanation of the important fields is provided underneath the diagram.

![image alt text](TD_-_Sequence_-_CreateOrUpdateCarePlan_-_Annotated.png)

```xml
<feed xmlns="http://www.w3.org/2005/Atom">
	<id>urn:uuid:3450807c-a1c4-4422-b898-1c3778244135</id>
	<category term="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Domain#HealthOnline" label="HealthOnline" scheme="http://hl7.org/fhir/tag/security" />
	<category term="http://hl7.org/fhir/tag/message" scheme="http://hl7.org/fhir/tag" />
	<entry>
		<content type="text/xml">
			<MessageHeader id="6c486493-f22d-4bf7-a5f3-e5e43ed5508c" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageHeader#Patient">
					<valueResource>
						<reference value="http://co.healthonline.nl/Patient/215325" />
					</valueResource>
				</extension>
				<identifier value="6c486493-f22d-4bf7-a5f3-e5e43ed5508c" />
				<timestamp value="2016-04-11T16:35:13+02:00" />
				<event>
					<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageEvents" />
					<code value="CreateOrUpdateCarePlan" />
					<display value="CreateOrUpdateCarePlan" />
				</event>
				<source>
					<software value="Koppeltaal Samples" />
					<endpoint value="Not used" />
				</source>
				<data>
					<reference value="http://co.healthonline.nl/CarePlan/32235/_history/2016-04-11T14:33:54:557.5771" />
				</data>
			</MessageHeader>
		</content>
	</entry>
	<entry>
		<id>http://co.healthonline.nl/CarePlan/32235</id>
		<link rel="self" href="http://co.healthonline.nl/CarePlan/32235/_history/2016-04-11T14:33:54:557.5771" />
		<content type="text/xml">
			<CarePlan id="32235" xmlns="http://hl7.org/fhir">
				<patient>
					<reference value="http://co.healthonline.nl/Patient/215325" />
				</patient>
				<status value="active" />
				<participant>
					<role>
						<coding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanParticipantRole" />
							<code value="Requester" />
							<display value="Requester" />
						</coding>
					</role>
					<member>
						<reference value="http://co.healthonline.nl/Practitioner/187534" />
					</member>
				</participant>
				<activity id="21232123523">
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityIdentifier">
						<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityDefinition">
						<valueString value="PVPGAME" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityKind">
						<valueCoding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/ActivityKind" />
							<code value="Game" />
							<display value="Game" />
						</valueCoding>
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#Participant">
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ParticipantMember">
							<valueResource>
								<reference value="http://co.healthonline.nl/Practitioner/187534" />
							</valueResource>
						</extension>
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ParticipantRole">
							<valueCodeableConcept>
								<coding>
									<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanParticipantRole" />
									<code value="Caregiver" />
									<display value="Caregiver" />
								</coding>
							</valueCodeableConcept>
						</extension>
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#StartDate">
						<valueDateTime value="2016-04-11T16:35:13+02:00" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityStatus">
						<valueCoding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
							<code value="Available" />
							<display value="Available" />
						</valueCoding>
					</extension>
					<prohibited value="false" />
					<simple>
						<category value="procedure" />
						<performer>
							<reference value="http://co.healthonline.nl/Patient/215325" />
						</performer>
					</simple>
				</activity>
			</CarePlan>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Patient/215325" />
		<content type="text/xml">
			<Patient id="215325" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Patient#Age">
					<valueInteger value="44" />
				</extension>
				<name>
					<use value="official" />
					<family value="Todea" />
					<given value="Reli" />
				</name>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
				<birthDate value="1972-02-28T00:00:00+01:00" />
			</Patient>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Practitioner/187534" />
		<content type="text/xml">
			<Practitioner id="187534" xmlns="http://hl7.org/fhir">
				<identifier>
					<use value="official" />
					<label value="BSN" />
					<system value="http://fhir.nl/fhir/NamingSystem/bsn" />
					<value value="309870665" />
				</identifier>
				<name>
					<use value="official" />
					<family value="Hole" />
					<given value="Harry" />
				</name>
				<telecom>
					<system value="email" />
					<value value="h.hole@spam.nl" />
					<use value="work" />
				</telecom>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
			</Practitioner>
		</content>
	</entry>
</feed>

```
A full description of all information contained in the message is beyond the scope of this document. A technical description of all fields of the CarePlan resource can be found here.

Do however note the ActivityIdentifier and ActivityDefinition:
```
...
<CarePlan>
...
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityIdentifier">
		<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
	</extension>
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityDefinition">
		<valueString value="PVPGAME" />
	</extension>
...
</CarePlan>
...
```

The ActivityIdentifier is intended to be used to refer to a specific care plan activity in the UpdateCarePlanActivityStatus and CreateOrUpdateCarePlanActivityResult messages. In this example it is constructed as '<CarePlanResourceID>#<internal activity number>', but this is not a requirement. Any string value is allowed.

The value of the ActivityDefinition field is the identifier of the intended ActivityDefinition. In this example the used value is the identifier of the Patients vs. Pollen game. (As described in Koppeltaal Server configuration).

Step 2: Patient starts activity
After the patient receives the invitation they will log in to the patient portal 'MyHealthOnline'. In the patient portal a link is available from which the patient can start the game, via the SSO sequence. The SSO sequence is out of the scope of this tutorial, see section 'Web Launch Sequence' in the technical design for more detail.

When Patients vs. Pollen is started the game sends the following UpdateCarePlanActivityStatus message.
```xml
<feed xmlns="http://www.w3.org/2005/Atom">
	<id>urn:uuid:1f8796c1-cb15-461e-ae79-752dc0d8de02</id>
	<category term="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Domain#HealthOnline" label="HealthOnline" scheme="http://hl7.org/fhir/tag/security" />
	<category term="http://hl7.org/fhir/tag/message" scheme="http://hl7.org/fhir/tag" />
	<entry>
		<content type="text/xml">
			<MessageHeader id="3259c4f2-4c61-4ef1-ab13-c4ddcda30976" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageHeader#Patient">
					<valueResource>
						<reference value="http://co.healthonline.nl/Patient/215325" />
					</valueResource>
				</extension>
				<identifier value="3259c4f2-4c61-4ef1-ab13-c4ddcda30976" />
				<timestamp value="2016-04-12T11:32:00+02:00" />
				<event>
					<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageEvents" />
					<code value="UpdateCarePlanActivityStatus" />
					<display value="UpdateCarePlanActivityStatus" />
				</event>
				<source>
					<software value="Koppeltaal Samples" />
					<endpoint value="Not used" />
				</source>
				<data>
					<reference value="http://pvp.tutorial.nl/CarePlanActivityStatus/8723212413224/_history/2016-04-12T07:45:27:889.2420" />
				</data>
			</MessageHeader>
		</content>
	</entry>
	<entry>
		<id>http://pvp.tutorial.nl/CarePlanActivityStatus/8723212413224</id>
		<link rel="self" href="http://pvp.tutorial.nl/CarePlanActivityStatus/8723212413224/_history/2016-04-12T07:45:27:889.2420" />
		<content type="text/xml">
			<Other xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#Activity">
					<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
				</extension>
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#PercentageCompleted">
					<valueDecimal value="10" />
				</extension>
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#ActivityStatus">
					<valueCoding>
						<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
						<code value="InProgress" />
						<display value="InProgress" />
					</valueCoding>
				</extension>
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#SubActivity">
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#SubActivityIdentifier">
						<valueString value="Level1" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#SubActivityStatus">
						<valueCoding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
							<code value="InProgress" />
							<display value="InProgress" />
						</valueCoding>
					</extension>
				</extension>
				<code>
					<coding>
						<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/OtherResourceUsage" />
						<code value="CarePlanActivityStatus" />
						<display value="CarePlanActivityStatus" />
					</coding>
				</code>
			</Other>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Patient/215325" />
		<content type="text/xml">
			<Patient id="215325" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Patient#Age">
					<valueInteger value="44" />
				</extension>
				<name>
					<use value="official" />
					<family value="Doe" />
					<given value="John" />
				</name>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
				<birthDate value="1972-02-28T00:00:00+01:00" />
			</Patient>
		</content>
	</entry>
</feed>
```

The details of the UpdateActivityStatus message can be found in the Technical Design. Note the following points of interest:

The UpdateCarePlanActivityStatus message is sent in the context of the same patient as the CarePlan message.
```xml
...
<MessageHeader>
...
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageHeader#Patient">
		<valueResource>
			<reference value="http://co.healthonline.nl/Patient/215325" />
		</valueResource>
	</extension>
...
</MessageHeader>
...
```
The status update resource refers to the care plan activity using the ActivityIdentifier.
```xml
...
<Other>
...
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus#Activity">
		<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
	</extension>
...
<Other>
...
```

To indicate that the patient is currently playing the first level, a sub activity has been added to the activity:
```xml
<Other>
	...
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivity">
		<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityIdentifier">
			<valueString value="Level1" />
		</extension>
		<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityStatus">
			<valueCoding>
				<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
				<code value="InProgress" />
				<display value="InProgress" />
			</valueCoding>
		</extension>
	</extension>
	...
</Other>
```

Because only the practitioner portal is subscribed to UpdateCarePlanActivityStatus messages the CaregiversOnline portal is the only one to receive a copy of this message. The practitioner portal will update the care plan based on the new status, as described in the next section.

Step 3: Practitioner portal updates CarePlan
When the practitioner portal 'CaregiversOnline' receives an UpdateCarePlanActivityStatus message it processes the message and applies the changes to the relevant activity. In the case of this example the change in activity status has no further consequences. In a real-life situation however there may be more than one activity in the CarePlan. In such a case it it possible for relations between activities to exist. There are situations in which a specific activity can only be started after a preceding activity has been completed. For example, certain interventions may only be started after the patient has first completed a ROM questionnaire to record their 'before' score.

The update of the CarePlan is sent to the koppeltaal Server by the practitioner portal via a CreateOrUpdateCarePlan message.
```xml
<feed xmlns="http://www.w3.org/2005/Atom">
	<id>urn:uuid:bb46b4e8-96d3-49ea-953e-78fcd36cbe72</id>
	<category term="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Domain#HealthOnline" label="HealthOnline" scheme="http://hl7.org/fhir/tag/security" />
	<category term="http://hl7.org/fhir/tag/message" scheme="http://hl7.org/fhir/tag" />
	<entry>
		<content type="text/xml">
			<MessageHeader id="46744a05-a837-4669-b869-1037f9db1ea1" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageHeader#Patient">
					<valueResource>
						<reference value="http://co.healthonline.nl/Patient/215325" />
					</valueResource>
				</extension>
				<identifier value="46744a05-a837-4669-b869-1037f9db1ea1" />
				<timestamp value="2016-04-12T11:34:06+02:00" />
				<event>
					<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageEvents" />
					<code value="CreateOrUpdateCarePlan" />
					<display value="CreateOrUpdateCarePlan" />
				</event>
				<source>
					<software value="Koppeltaal Samples" />
					<endpoint value="Not used" />
				</source>
				<data>
					<reference value="http://co.healthonline.nl/CarePlan/32235/_history/2016-04-12T09:30:23:889.9204" />
				</data>
			</MessageHeader>
		</content>
	</entry>
	<entry>
		<id>http://co.healthonline.nl/CarePlan/32235</id>
		<link rel="self" href="http://co.healthonline.nl/CarePlan/32235/_history/2016-04-12T09:30:23:889.9204" />
		<content type="text/xml">
			<CarePlan id="32235" xmlns="http://hl7.org/fhir">
				<patient>
					<reference value="http://co.healthonline.nl/Patient/215325" />
				</patient>
				<status value="active" />
				<participant>
					<role>
						<coding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanParticipantRole" />
							<code value="Requester" />
							<display value="Requester" />
						</coding>
					</role>
					<member>
						<reference value="http://co.healthonline.nl/Practitioner/187534" />
					</member>
				</participant>
				<activity id="21232123523">
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityIdentifier">
						<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityDefinition">
						<valueString value="PVPGAME" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityKind">
						<valueCoding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/ActivityKind" />
							<code value="Game" />
							<display value="Game" />
						</valueCoding>
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivity">
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityIdentifier">
							<valueString value="Level1" />
						</extension>
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityStatus">
							<valueCoding>
								<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
								<code value="InProgress" />
								<display value="InProgress" />
							</valueCoding>
						</extension>
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#Participant">
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ParticipantMember">
							<valueResource>
								<reference value="http://co.healthonline.nl/Practitioner/187534" />
							</valueResource>
						</extension>
						<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ParticipantRole">
							<valueCodeableConcept>
								<coding>
									<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanParticipantRole" />
									<code value="Caregiver" />
									<display value="Caregiver" />
								</coding>
							</valueCodeableConcept>
						</extension>
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#StartDate">
						<valueDateTime value="2016-04-12T11:34:06+02:00" />
					</extension>
					<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityStatus">
						<valueCoding>
							<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
							<code value="InProgress" />
							<display value="InProgress" />
						</valueCoding>
					</extension>
					<prohibited value="false" />
					<simple>
						<category value="procedure" />
						<performer>
							<reference value="http://co.healthonline.nl/Patient/215325" />
						</performer>
					</simple>
				</activity>
			</CarePlan>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Patient/215325" />
		<content type="text/xml">
			<Patient id="215325" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Patient#Age">
					<valueInteger value="44" />
				</extension>
				<name>
					<use value="official" />
					<family value="Doe" />
					<given value="John" />
				</name>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
				<birthDate value="1972-02-28T00:00:00+01:00" />
			</Patient>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Practitioner/187534" />
		<content type="text/xml">
			<Practitioner id="187534" xmlns="http://hl7.org/fhir">
				<identifier>
					<use value="official" />
					<label value="BSN" />
					<system value="http://fhir.nl/fhir/NamingSystem/bsn" />
					<value value="309870665" />
				</identifier>
				<name>
					<use value="official" />
					<family value="Hole" />
					<given value="Harry" />
				</name>
				<telecom>
					<system value="email" />
					<value value="h.hole@spam.nl" />
					<use value="work" />
				</telecom>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
			</Practitioner>
		</content>
	</entry>
</feed>
```

In this case, the only changes are the status of the activity and the addition of the sub activity.
```xml
...
<CarePlan>
	...
	<activity>
		...
		<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#ActivityStatus">
			<valueCoding>
				<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
				<code value="InProgress" />
				<display value="InProgress" />
			</valueCoding>
		</extension>
		...
		<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivity">
			<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityIdentifier">
				<valueString value="Level1" />
			</extension>
			<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlan#SubActivityStatus">
				<valueCoding>
					<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityStatus" />
					<code value="InProgress" />
					<display value="InProgress" />
				</valueCoding>
			</extension>
		</extension>
		...
	</activity>
	...
</CarePlan>
...
```
Since both the patient portal and the game are subscribed to messages of type 'CreateOrUpdateCarePlan' both will receive a copy. The patient portal will process the message, perhaps resulting in a different way of displaying the link to the Patients vs. Pollen game to provide a visual reminder that it is in progress. The game was the original sender of the change, so should detect no changes to the activity.

Step 4: Patient completes part of the care plan activity[edit]
As the patient progresses through the game (intermediary) results may become available. Let's assume that in the case of Patients vs. Pollen the patient is presented with questions about their allergies in between levels. Based on the answers to these questions and the results of their game play various metrics are computed.

When the metrics become available (or are updated) the game sends a CreateOrUpdateCarePlanActivityResult message. This message may look as follows.
```xml
<feed xmlns="http://www.w3.org/2005/Atom">
	<id>urn:uuid:7d577b39-5463-4cde-b7e5-3fc62600d7fe</id>
	<category term="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Domain#HealthOnline" label="HealthOnline" scheme="http://hl7.org/fhir/tag/security" />
	<category term="http://hl7.org/fhir/tag/message" scheme="http://hl7.org/fhir/tag" />
	<entry>
		<content type="text/xml">
			<MessageHeader id="60b59816-1b6d-401b-a478-b42f2a4017ad" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageHeader#Patient">
					<valueResource>
						<reference value="http://co.healthonline.nl/Patient/215325" />
					</valueResource>
				</extension>
				<identifier value="60b59816-1b6d-401b-a478-b42f2a4017ad" />
				<timestamp value="2016-04-12T10:58:54+02:00" />
				<event>
					<system value="http://ggz.koppeltaal.nl/fhir/Koppeltaal/MessageEvents" />
					<code value="CreateOrUpdateCarePlanActivityResult" />
					<display value="CreateOrUpdateCarePlanActivityResult" />
				</event>
				<source>
					<software value="Koppeltaal Samples" />
					<endpoint value="Not used" />
				</source>
				<data>
					<reference value="http://pvp.tutorial.nl/DiagnosticReport/11242451121342254/_history/2016-04-12T08:57:27:264.8844" />
				</data>
			</MessageHeader>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://pvp.tutorial.nl/DiagnosticReport/11242451121342254/_history/2016-04-12T08:57:27:264.8844" />
		<content type="text/xml">
			<DiagnosticReport id="11242451121342254" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityResult#Activity">
					<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
				</extension>
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityResult#Questionnaire">
					<valueResource>
						<reference value="http://pvp.tutorial.nl/Questionnaire/1421253" />
					</valueResource>
				</extension>
				<name>
					<text value="HADS40" />
				</name>
				<status value="final" />
				<issued value="2014-01" />
				<subject>
					<reference value="http://co.healthonline.nl/Patient/215325" />
				</subject>
				<performer>
					<reference value="http://pvp.tutorial.nl/Practitioner/982800" />
				</performer>
				<diagnosticDateTime value="2016-04-12T10:58:54+02:00" />
				<result>
					<reference value="12442" />
				</result>
			</DiagnosticReport>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://pvp.tutorial.nl/Questionnaire/1421253" />
		<content type="text/xml">
			<Questionnaire id="1421253" xmlns="http://hl7.org/fhir">
				<status value="in progress" />
				<authored value="Hl7.Fhir.Model.FhirDateTime" />
				<name>
					<text value="HADS4" />
				</name>
				<group>
					<question>
						<name>
							<coding>
								<system value="TutorialSystem" />
								<code value="HADS4.4" />
								<display value="HADS4.4" />
							</coding>
						</name>
						<text value="How many days per week do you suffer from allergies?" />
						<answerDecimal value="3" />
					</question>
				</group>
			</Questionnaire>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://pvp.tutorial.nl/Observation/12442" />
		<content type="text/xml">
			<Observation id="12442" xmlns="http://hl7.org/fhir">
				<name>
					<text value="Severity" />
				</name>
				<valueDecimal value="4" />
				<status value="final" />
				<reliability value="ok" />
			</Observation>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://co.healthonline.nl/Patient/215325" />
		<content type="text/xml">
			<Patient id="215325" xmlns="http://hl7.org/fhir">
				<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/Patient#Age">
					<valueInteger value="44" />
				</extension>
				<name>
					<use value="official" />
					<family value="Doe" />
					<given value="John" />
				</name>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
				<birthDate value="1972-02-28T00:00:00+01:00" />
			</Patient>
		</content>
	</entry>
	<entry>
		<link rel="self" href="http://pvp.tutorial.nl/Practitioner/982800" />
		<content type="text/xml">
			<Practitioner id="982800" xmlns="http://hl7.org/fhir">
				<identifier>
					<use value="official" />
					<label value="BSN" />
					<system value="http://fhir.nl/fhir/NamingSystem/bsn" />
					<value value="309870665" />
				</identifier>
				<name>
					<use value="official" />
					<family value="Hole" />
					<given value="Harry" />
				</name>
				<telecom>
					<system value="email" />
					<value value="h.hole@spam.nl" />
					<use value="work" />
				</telecom>
				<gender>
					<coding>
						<system value="http://hl7.org/fhir/vs/administrative-gender" />
						<code value="M" />
						<display value="Male" />
					</coding>
				</gender>
			</Practitioner>
		</content>
	</entry>
</feed>
```
The exact details of the implementation of the result can be found in the technical design.

Note again a reference to the care plan activity:
```xml
...
<DiagnosticReport>
	...
	<extension url="http://ggz.koppeltaal.nl/fhir/Koppeltaal/CarePlanActivityResult#Activity">
		<valueString value="http://co.healthonline.nl/CarePlan/32235#21232123523" />
	</extension>
	...
<DiagnosticReport>
...
```
Since both the patient and practitioners portals are subscribed to messages of type CreateOrUpdateCarePlanActivityResult they both receive a copy of the message. The patient portal may provide the patient with an overview of their played levels and the answers given while the practitioner portal may provide more detailed statistics and the calculated scores for the practitioner to peruse.

Step 5 & 6: Patient continues playing the game
As the patient continues playing the game, scoring points and answering questions the results of the game may change. At some point the patient will reach the final level and complete the game. The Patients vs. Pollen game will continue to send out CreateUrUpdateCarePlanActivityResult and UpdateCarePlanActivityStatus messages.

Messages 5 and 6 in the diagram are very similar to the messages of steps 2 and 3 described earlier on this page.

