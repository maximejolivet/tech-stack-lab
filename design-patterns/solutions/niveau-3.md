# Solutions — Niveau 3 : Avancé

## 1. Dependency Inversion et testabilité

Dépendances couplées en dur : `PDO` (base de données MySQL précise) et `PHPMailer` (librairie d'envoi précise).

```php
interface OrderRepositoryInterface {
    public function save(array $items): void;
}
interface MailerInterface {
    public function send(string $to, string $subject, string $body): void;
}

class OrderService {
    public function __construct(
        private OrderRepositoryInterface $repository,
        private MailerInterface $mailer,
    ) {}

    public function placeOrder(array $items): void {
        $this->repository->save($items);
        $this->mailer->send('customer@example.com', 'Confirmation', 'Votre commande est confirmée');
    }
}

// Implémentations en mémoire pour les tests
class InMemoryOrderRepository implements OrderRepositoryInterface {
    public array $saved = [];
    public function save(array $items): void { $this->saved[] = $items; }
}
class FakeMailer implements MailerInterface {
    public array $sent = [];
    public function send(string $to, string $subject, string $body): void {
        $this->sent[] = compact('to', 'subject', 'body');
    }
}

// Test
$repository = new InMemoryOrderRepository();
$mailer = new FakeMailer();
$service = new OrderService($repository, $mailer);

$service->placeOrder(['item-1', 'item-2']);

assert(count($repository->saved) === 1);
assert(count($mailer->sent) === 1);
```

Aucune connexion réseau ni base réelle nécessaire : le test s'exécute instantanément et vérifie le comportement, pas l'infrastructure.

## 2. Composition vs héritage

```js
const CodingBehavior = { code() { return "code"; } };
const ManagingBehavior = { manage() { return "manage une équipe"; } };
const BillingBehavior = { bill() { return "facture le client"; } };

function createEmployee(...behaviors) {
  return Object.assign({}, ...behaviors);
}

const developer = createEmployee(CodingBehavior);
const manager = createEmployee(ManagingBehavior);
const techLead = createEmployee(CodingBehavior, ManagingBehavior);
const freelancer = createEmployee(CodingBehavior, ManagingBehavior, BillingBehavior);
```

N'importe quelle combinaison de capacités devient possible sans créer une nouvelle classe par combinaison — le problème combinatoire de l'héritage (une classe par cas) disparaît au profit d'un assemblage à la demande.

## 3. Repository + Factory

```
NotificationChannelFactory::create(string $type): NotificationChannel
  → EmailChannel | SmsChannel | PushChannel (chacune implements NotificationChannel::send())

NotificationRepositoryInterface::log(Notification $n): void
  → SqlNotificationRepository | InMemoryNotificationRepository

NotificationService (orchestrateur) :
  - reçoit NotificationRepositoryInterface et NotificationChannelFactory par constructeur
  - send(type, message): crée le canal via la Factory, envoie, puis logue via le Repository
```

Justification (5 lignes) : chaque classe a une seule raison de changer (SRP) — la Factory ne change que si un nouveau canal apparaît, le Repository ne change que si le mode de stockage change, un canal ne change que si son API d'envoi change. Chaque brique se teste isolément (repository en mémoire, canal fake) sans dépendre des autres. Remplacer le stockage SQL par un stockage fichier ne touche ni les canaux ni la Factory. Ajouter un canal Slack ne touche ni le Repository ni le NotificationService. Une classe unique `NotificationManager` couplerait ces trois responsabilités : un changement sur l'un des trois axes forcerait à retoucher (et retester) l'ensemble.

## 4. Reconnaître la sur-ingénierie

Éléments probablement en trop pour un formulaire à 3 champs et un seul traitement possible :
- **`ContactFormFactory` + `ContactFormStrategy`** : n'a de sens que s'il existe (ou est concrètement prévu à court terme) plusieurs types de traitement. Avec un seul type (`EmailContactFormStrategy`), c'est de l'indirection sans bénéfice — violerait YAGNI et la Rule of Three (une seule occurrence, pas trois).
- **`ContactFormObserver` avec 3 listeners vides** : événements sans aucun abonné réel = complexité pure, à ajouter le jour où un vrai besoin de notification apparaît.
- **`ContactFormValidatorChainOfResponsibility`** : une Chain of Responsibility se justifie si les validateurs doivent être réordonnés/désactivés dynamiquement ; pour 3 champs simples, une fonction de validation classique (ou une poignée d'`if`) suffit et reste plus lisible.

Ce qui a du sens à garder : le **`ContactFormRepository`** si la persistance avant envoi est un vrai besoin métier (traçabilité, audit) — mais seulement dans ce cas, pas par principe. Le reste peut être réintroduit plus tard, quand un deuxième cas d'usage concret apparaît (cf. Rule of Three), plutôt que d'être anticipé aujourd'hui.
