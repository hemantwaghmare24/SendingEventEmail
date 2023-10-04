from rest_framework import  status
from rest_framework.response import Response
from django.core.mail import send_mail
from datetime import date
from rest_framework.generics import CreateAPIView,
from .models import  Event, Employee, Emailtemplate, Emaillog
from .serializers import EventSerializer, EmployeeSerializer, EmailtemplateSerializer, EmaillogSerializer

class EmailSenderView(CreateAPIView):
    def send_emails(self, request):
        try:
            todaysdate = date.today()
            Events = Event.objects.filter(date= todaysdate)
            if not Events.exists():
                EmailLog.objects.create(Status='No event scheduled')
                return Response({'message': 'No events scheduled for Today'}, status=status.HTTP_200_OK)
            
            for event in Events:
                EventSerializeddata = EventSerializer(event).data
                EventType = EventSerializeddata ['eventtype']
                
                # Fetch email template based on event type
                emailtemplatequery = Emailtemplate.objects.get(eventtype= EventType)
                emailcontent details= emailtemplatequery.content
                
                # Fetch employees for the event
                Employeequery = Employee.objects.filter(event=event)
                
                for employee in Employeequery:
                    # Populate email template with employee details and event-specific content
                    emailbody = email_content.format(employeename=employee.name, eventname= EventSerializeddata ['name'])
                    
                    # Send the personalized email
                    send_mail(
                        subject=f"Event Reminder: { EventSerializeddata ['name']}",
                        message=emailbody,
                        from_email='your@email.com',
                        recipient_list=[employee.email],
                        fail_silently=False,
                    )
                    
                    # Log email sending status
                    EmailLog.objects.create(
                        employee=employee,
                        event=event,
                        status='Sent',
                    )
            
            return Response({'message': 'Emails sent successfully'}, status=status.HTTP_200_OK)
        except Exception as e:
            # Log the error and continue with the next scheduled email
            EmailLog.objects.create(
                employee=None,
                event=None,
                status=f'Error: {str(e)}',
            )
            return Response({'message': 'Email sending failed'}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
