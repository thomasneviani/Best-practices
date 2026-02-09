Voici une implémentation complète du pattern Result pour Symfony avec DDD :

## Classe Result de base

```php
<?php

namespace App\Domain\Shared;

readonly class Result
{
    private function __construct(
        private bool $isSuccess,
        private mixed $value = null,
        private ?Error $error = null
    ) {}

    public static function success(mixed $value = null): self
    {
        return new self(true, $value, null);
    }

    public static function failure(Error $error): self
    {
        return new self(false, null, $error);
    }

    public function isSuccess(): bool
    {
        return $this->isSuccess;
    }

    public function isFailure(): bool
    {
        return !$this->isSuccess;
    }

    public function getValue(): mixed
    {
        if (!$this->isSuccess) {
            throw new \LogicException('Cannot get value from failed result');
        }
        return $this->value;
    }

    public function getError(): Error
    {
        if ($this->isSuccess) {
            throw new \LogicException('Cannot get error from successful result');
        }
        return $this->error;
    }
}
```

## Classe Error

```php
<?php

namespace App\Domain\Shared;

readonly class Error
{
    public function __construct(
        private string $code,
        private string $message,
        private array $details = []
    ) {}

    public function getCode(): string
    {
        return $this->code;
    }

    public function getMessage(): string
    {
        return $this->message;
    }

    public function getDetails(): array
    {
        return $this->details;
    }

    public static function validation(string $message, array $details = []): self
    {
        return new self('VALIDATION_ERROR', $message, $details);
    }

    public static function notFound(string $message): self
    {
        return new self('NOT_FOUND', $message);
    }

    public static function conflict(string $message): self
    {
        return new self('CONFLICT', $message);
    }
}
```

## Exemple d'utilisation dans un service Domain

```php
<?php

namespace App\Domain\Order\Service;

use App\Domain\Order\Entity\Order;
use App\Domain\Order\Repository\OrderRepositoryInterface;
use App\Domain\Order\ValueObject\OrderId;
use App\Domain\Shared\Result;
use App\Domain\Shared\Error;

final readonly class OrderValidator
{
    public function __construct(
        private OrderRepositoryInterface $orderRepository
    ) {}

    public function validateOrderForShipment(OrderId $orderId): Result
    {
        $order = $this->orderRepository->findById($orderId);
        
        if (!$order) {
            return Result::failure(
                Error::notFound("Order {$orderId->toString()} not found")
            );
        }

        if (!$order->isPaid()) {
            return Result::failure(
                Error::validation('Order must be paid before shipment')
            );
        }

        if ($order->hasBeenShipped()) {
            return Result::failure(
                Error::conflict('Order has already been shipped')
            );
        }

        if ($order->getItems()->isEmpty()) {
            return Result::failure(
                Error::validation('Order must contain at least one item')
            );
        }

        return Result::success($order);
    }
}
```

## Utilisation dans un Command Handler

```php
<?php

namespace App\Application\Order\CommandHandler;

use App\Application\Order\Command\ShipOrderCommand;
use App\Domain\Order\Service\OrderValidator;
use App\Domain\Order\Repository\OrderRepositoryInterface;
use App\Domain\Shared\Result;

final readonly class ShipOrderCommandHandler
{
    public function __construct(
        private OrderValidator $orderValidator,
        private OrderRepositoryInterface $orderRepository
    ) {}

    public function __invoke(ShipOrderCommand $command): Result
    {
        $validationResult = $this->orderValidator->validateOrderForShipment(
            $command->getOrderId()
        );

        if ($validationResult->isFailure()) {
            return $validationResult;
        }

        $order = $validationResult->getValue();
        
        try {
            $order->markAsShipped();
            $this->orderRepository->save($order);
            
            return Result::success($order);
        } catch (\Exception $e) {
            // Exceptions techniques restent des exceptions
            throw new \RuntimeException('Failed to ship order', 0, $e);
        }
    }
}
```

## Gestion dans le Controller

```php
<?php

namespace App\Infrastructure\Controller;

use App\Application\Order\Command\ShipOrderCommand;
use App\Application\Order\CommandHandler\ShipOrderCommandHandler;
use App\Domain\Order\ValueObject\OrderId;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

final class OrderController extends AbstractController
{
    #[Route('/orders/{id}/ship', methods: ['POST'])]
    public function shipOrder(
        string $id,
        ShipOrderCommandHandler $handler
    ): JsonResponse {
        $command = new ShipOrderCommand(new OrderId($id));
        $result = $handler($command);

        if ($result->isFailure()) {
            $error = $result->getError();
            
            $statusCode = match ($error->getCode()) {
                'NOT_FOUND' => Response::HTTP_NOT_FOUND,
                'VALIDATION_ERROR' => Response::HTTP_BAD_REQUEST,
                'CONFLICT' => Response::HTTP_CONFLICT,
                default => Response::HTTP_INTERNAL_SERVER_ERROR,
            };

            return $this->json([
                'error' => [
                    'code' => $error->getCode(),
                    'message' => $error->getMessage(),
                    'details' => $error->getDetails(),
                ]
            ], $statusCode);
        }

        $order = $result->getValue();
        
        return $this->json([
            'id' => $order->getId()->toString(),
            'status' => 'shipped'
        ], Response::HTTP_OK);
    }
}
```

Cette implémentation  suit les principes DDD en gardant les erreurs métier explicites dans le domaine, tout en permettant aux exceptions techniques de remonter naturellement. Le pattern Result force les appelants à gérer consciemment les cas d'échec métier, rendant le code plus robuste et prévisible.[1][2]

Sources
[1] Result<T, E> type in PHP https://dev.to/crusty0gphr/resultt-e-type-in-php-3dab
[2] Domain Driven Design with PHP and Symfony https://dev.to/ludofleury/domain-driven-design-with-php-and-symfony-1bl6
[3] I wrote a library implementing the Option & Result types in ... https://www.reddit.com/r/PHP/comments/wtivjz/i_wrote_a_library_implementing_the_option_result/
[4] Strategy in PHP / Design Patterns https://refactoring.guru/design-patterns/strategy/php/example
[5] A PHP Pattern To Avoid Try/Catch Blocks Repetition https://betterprogramming.pub/a-php-pattern-to-avoid-try-catch-blocks-repetition-1e3fe2038dc1
[6] PHP: rfc:pattern-matching https://wiki.php.net/rfc/pattern-matching
[7] Creating Symfony services using the factory pattern | https://www.ewaldvanderveken.dev/creating-symfony-services-using-the-factory-pattern/
[8] ResultPattern package https://www.reddit.com/r/dotnet/comments/1gbqamp/resultpattern_package/
[9] Domain Driven Design avec PHP & Symfony https://dev.to/ludofleury/domain-driven-design-avec-php-symfony-1p2h
[10] Write a class in PHP Symfony 4.4 using the design pattern Factory Method to instantiate Service classes https://stackoverflow.com/questions/59085530/write-a-class-in-php-symfony-4-4-using-the-design-pattern-factory-method-to-inst
