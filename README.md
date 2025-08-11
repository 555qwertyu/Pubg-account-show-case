# pubg_marketplace/settings.py
INSTALLED_APPS = [
    # ...
    'django.contrib.humanize',
    'accounts',
    'products',
    'transactions',
]

# accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    is_seller = models.BooleanField(default=False)
    is_buyer = models.BooleanField(default=True)
    balance = models.DecimalField(max_digits=10, decimal_places=2, default=0)

# products/models.py
from django.db import models
from django.conf import settings

class PUBGAccount(models.Model):
    RANK_CHOICES = [
        ('Bronze', 'Bronze'),
        ('Silver', 'Silver'),
        ('Gold', 'Gold'),
        ('Platinum', 'Platinum'),
        ('Diamond', 'Diamond'),
        ('Crown', 'Crown'),
        ('Ace', 'Ace'),
        ('Conqueror', 'Conqueror'),
    ]
    
    seller = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rank = models.CharField(max_length=20, choices=RANK_CHOICES)
    level = models.IntegerField()
    skins_count = models.IntegerField()
    rare_items = models.TextField()
    region = models.CharField(max_length=50)
    created_at = models.DateTimeField(auto_now_add=True)
    is_sold = models.BooleanField(default=False)
    # Automatic delivery fields
    login_email = models.EmailField(blank=True)
    login_password = models.CharField(max_length=200, blank=True)

    def __str__(self):
        return self.title

# transactions/models.py
from django.db import models
from django.conf import settings
from products.models import PUBGAccount

class Order(models.Model):
    buyer = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    account = models.ForeignKey(PUBGAccount, on_delete=models.CASCADE)
    amount = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    payment_id = models.CharField(max_length=100)
    is_completed = models.BooleanField(default=False)

# transactions/views.py
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from products.models import PUBGAccount
from .models import Order
import stripe
from django.conf import settings

stripe.api_key = settings.STRIPE_SECRET_KEY

@login_required
def purchase_account(request, account_id):
    account = get_object_or_404(PUBGAccount, id=account_id, is_sold=False)
    
    if request.method == 'POST':
        try:
            # Create Stripe charge
            charge = stripe.Charge.create(
                amount=int(account.price * 100),
                currency='usd',
                source=request.POST['stripeToken'],
                description=f'Purchase of PUBG account: {account.title}'
            )
            
            # Create order record
            order = Order.objects.create(
                buyer=request.user,
                account=account,
                amount=account.price,
                payment_id=charge.id,
                is_completed=True
            )
            
            # Mark account as sold and prepare for delivery
            account.is_sold = True
            account.save()
            
            # Automatic delivery (in real app, send email instead)
            request.session['purchased_account'] = {
                'email': account.login_email,
                'password': account.login_password
            }
            
            return redirect('order_success')
            
        except stripe.error.StripeError as e:
            return render(request, 'error.html', {'error': str(e)})
    
    return render(request, 'checkout.html', {
        'account': account,
        'stripe_public_key': settings.STRIPE_PUBLIC_KEY
    })

def order_success(request):
    account_details = request.session.pop('purchased_account', None)
    return render(request, 'order_success.html', {'account_details': account_details})

# products/views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from .models import PUBGAccount
from .forms import PUBGAccountForm

def marketplace(request):
    accounts = PUBGAccount.objects.filter(is_sold=False).order_by('-created_at')
    return render(request, 'marketplace.html', {'accounts': accounts})

@login_required
def create_listing(request):
    if request.method == 'POST':
        form = PUBGAccountForm(request.POST)
        if form.is_valid():
            account = form.save(commit=False)
            account.seller = request.user
            account.save()
            return redirect('marketplace')
    else:
        form = PUBGAccountForm()
    return render(request, 'create_listing.html', {'form': form})

# transactions/webhooks.py (for automatic payment confirmation)
import stripe
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from .models import Order

@csrf_exempt
def stripe_webhook(request):
    payload = request.body
    sig_header = request.META['HTTP_STRIPE_SIGNATURE']
    event = None

    try:
        event = stripe.Webhook.construct_event(
            payload, sig_header, settings.STRIPE_WEBHOOK_SECRET
        )
    except ValueError:
        return HttpResponse(status=400)
    except stripe.error.SignatureVerificationError:
        return HttpResponse(status=400)

    if event['type'] == 'checkout.session.completed':
        session = event['data']['object']
        # Handle successful payment

    return HttpResponse(status=200)Traceback (most recent call last):
  File "/data/user/0/ru.iiec.pydroid3/files/accomp_files/iiec_run/iiec_run.py", line 31, in <module>
    start(fakepyfile,mainpyfile)
    ~~~~~^^^^^^^^^^^^^^^^^^^^^^^
  File "/data/user/0/ru.iiec.pydroid3/files/accomp_files/iiec_run/iiec_run.py", line 30, in start
    exec(open(mainpyfile).read(),  __main__.__dict__)
    ~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "<string>", line 11, in <module>
ModuleNotFoundError: No module named 'django'

[Program finished]Traceback (most recent call last):
  File "/data/user/0/ru.iiec.pydroid3/files/accomp_files/iiec_run/iiec_run.py", line 31, in <module>
    start(fakepyfile,mainpyfile)
    ~~~~~^^^^^^^^^^^^^^^^^^^^^^^
  File "/data/user/0/ru.iiec.pydroid3/files/accomp_files/iiec_run/iiec_run.py", line 30, in start
    exec(open(mainpyfile).read(),  __main__.__dict__)
    ~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "<string>", line 11, in <module>
ModuleNotFoundError: No module named 'django'

[Program finished]
